---
layout: post
title:  "Chromium C++与JS之间互调"
date:  2015-02-01
categories: Chromium
featured_image: /images/cover.jpg
---

###Chromium C++与JS之间互调

####C++调用JS函数

![](https://github.com/yang7229693/yang7229693.github.io/blob/master/images/webkit_c_js.png)

如图所示，在browser里，通过IPC，让render去调用webkit中相关函数来执行js代码，然后将执行结果通过IPC，从render返回给browser，这个过程就是chromium中c++调用js函数的一个粗略的过程吧，下面通过代码来看下是如何执行的吧。


c++调用js函数调用的是CallJavascriptFunction方法，在src/content/public/browser/web_ui.h里面定义了一系列的方法，如下所示：
	
	virtual void CallJavascriptFunction(const std::string& function_name) = 0;
	virtual void CallJavascriptFunction(const std::string& function_name,
                                        const base::Value& arg) = 0;
	virtual void CallJavascriptFunction(const std::string& function_name,
                                        const base::Value& arg1,
                                        const base::Value& arg2) = 0;
	virtual void CallJavascriptFunction(const std::string& function_name,
                                        const base::Value& arg1,
                                        const base::Value& arg2,
                                        const base::Value& arg3) = 0;
	virtual void CallJavascriptFunction(const std::string& function_name,
                                        const base::Value& arg1,
                                        const base::Value& arg2,
                                        const base::Value& arg3,
                                        const base::Value& arg4) = 0;
	virtual void CallJavascriptFunction(const std::string& function_name,
      									 const std::vector<const base::Value*>& args) = 0;

具体的实现在src/content/browser/webui/web_ui_impl中，例如最简单的调用无参数的js函数的实现如下:
	
	void WebUIImpl::CallJavascriptFunction(const std::string& function_name) {
	  DCHECK(base::IsStringASCII(function_name));
	  base::string16 javascript = base::ASCIIToUTF16(function_name + "();");
	  ExecuteJavascript(javascript);
	}

可以看出，该方法只是简单对函数名进行了处理，添加了一个圆括号，丢给ExecuteJavascript函数去执行，再继续深入代码可以看到，在src/content/browser/frame_host/render_frame_host_impl中有该方法的具体实现，如下：

	void RenderFrameHostImpl::ExecuteJavaScript(
    	const base::string16& javascript) {
		Send(new FrameMsg_JavaScriptExecuteRequest(routing_id_,
                                             javascript,
                                             0, false));
	}

可以看到是通过消息来进行通讯的，将需要执行的函数通过消息发送，在src/content/render/render_frame_impl中查看消息的响应函数，如下：

	IPC_MESSAGE_HANDLER(FrameMsg_JavaScriptExecuteRequest,
                        OnJavaScriptExecuteRequest)

	void RenderFrameImpl::OnJavaScriptExecuteRequest(
    	const base::string16& jscript,
    	int id,
    	bool notify_result) {
	TRACE_EVENT_INSTANT0("test_tracing", "OnJavaScriptExecuteRequest",
                       TRACE_EVENT_SCOPE_THREAD);

	v8::HandleScope handle_scope(v8::Isolate::GetCurrent());
	v8::Handle<v8::Value> result =
      	frame_->executeScriptAndReturnValue(WebScriptSource(jscript));

	HandleJavascriptExecutionResult(jscript, id, notify_result, result);	
	}

可以看到，处理过程是从browser到render，最后到webkit，这个就是一个单向的执行js过程，上面函数标红部分则是回调过程，当webkit里面执行完js函数后，然后通过IPC返回给browser来告知执行结果，在src/content/browser/frame_host/render_frame_host_impl中如下：

	IPC_MESSAGE_HANDLER(FrameHostMsg_JavaScriptExecuteResponse,
                        OnJavaScriptExecuteResponse)

	void RenderFrameHostImpl::OnJavaScriptExecuteResponse(
    	int id, const base::ListValue& result) {
	  const base::Value* result_value;
	  if (!result.Get(0, &result_value)) {
      // Programming error or rogue renderer.
	  NOTREACHED() << "Got bad arguments for OnJavaScriptExecuteResponse";
	  return;
	}

	  std::map<int, JavaScriptResultCallback>::iterator it =
	      javascript_callbacks_.find(id);
	  if (it != javascript_callbacks_.end()) {
	    it->second.Run(result_value);
	    javascript_callbacks_.erase(it);
	  } else {
	    NOTREACHED() << "Received script response for unknown request";
	  }
	}

executeScriptAndReturnValue方法在src/third_party/Webkit/Source/web/WebLocalFrameImpl中，v8是如何去执行js代码的，感兴趣的同学可以自己扒一下相关代码吧。


####JS函数调用C++代码

JS通过chrome.send函数来调用在C++中注册过的函数，大致过程跟C++调用JS函数类似吧，只不过是从webkit发起的，通过IPC从render到browser去执行对应的函数。直接上代码吧，在src/content/render/web_ui_extension.cc中：

	void WebUIExtension::Install(blink::WebFrame* frame) {
	  …
	  chrome->Set(gin::StringToSymbol(isolate, "send"),
	              gin::CreateFunctionTemplate(
	                  isolate, base::Bind(&WebUIExtension::Send))->GetFunction());
	  …

	void WebUIExtension::Send(gin::Arguments* args) {
	  …
	  render_view->Send(new ViewHostMsg_WebUISend(render_view->GetRoutingID(),
	                                              frame->document().url(),
	                                              message,
	                                              *content));
	  …

在install函数中为chrome.send绑定Send方法，在Send方法中，则通过消息来通知browser，接下来看下browser这边的处理吧，在src/content/browser/webui/web_ui_impl中：

	bool WebUIImpl::OnMessageReceived(const IPC::Message& message) {
	  …
	  IPC_MESSAGE_HANDLER(ViewHostMsg_WebUISend, OnWebUISend)
	  …

	void WebUIImpl::OnWebUISend(const GURL& source_url,
	                            const std::string& message,
	                            const base::ListValue& args) {
	  if (!ChildProcessSecurityPolicyImpl::GetInstance()->
	          HasWebUIBindings(web_contents_->GetRenderProcessHost()->GetID()) ||
	      !WebUIControllerFactoryRegistry::GetInstance()->IsURLAcceptableForWebUI(
	          web_contents_->GetBrowserContext(), source_url)) {
	    NOTREACHED() << "Blocked unauthorized use of WebUIBindings.";
	    return;
	  }

	  ProcessWebUIMessage(source_url, message, args);
	}

browser接受到消息后，通过ProcessWebUIMessage函数，在一个消息map中找到绑定的函数，然后执行其对应的函数，ProcessWebUIMessage函数如下：

	void WebUIImpl::ProcessWebUIMessage(const GURL& source_url,
	                                    const std::string& message,
	                                    const base::ListValue& args) {
	  if (controller_->OverrideHandleWebUIMessage(source_url, message, args))
	    return;

	  // Look up the callback for this message.
	  MessageCallbackMap::const_iterator callback =
	      message_callbacks_.find(message);
	  if (callback != message_callbacks_.end()) {
	    // Forward this message and content on.
	    callback->second.Run(&args);
	  } else {
	    NOTREACHED() << "Unhandled chrome.send(\"" << message << "\");";
	  }
	}