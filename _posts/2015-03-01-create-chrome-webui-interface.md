---
layout: post
title: 创建Chrome WebUI接口
categories: Chromium
excerpt: 创建chromium内置页以及对话框
tag: [google, chromium, webUI, interface, html]
comments: true
image:
  feature: pic-chrome-1.jpg
---

{% include _toc.html %}

#### 1 创建WebUI页面

WebUI的资源在src/chrome.browser/resources下面，当创建WebUI资源时，需要遵循[Web Development Style Guide](https://www.chromium.org/developers/web-development-style-guide) 下面是一个简单的例子：

添加html页面

{%highlight html%}
src/chrome/browser/resources/hello_world.html

<!DOCTYPE HTML>
<html i18n-values="dir:textdirection">
<head>
  <meta charset="utf-8">
  <title i18n-content="helloWorldTitle"></title>
  <link rel="stylesheet" href="hello_world.css">
  <script src="chrome://resources/js/cr.js"></script>
  <script src="chrome://resources/js/load_time_data.js"></script>
  <script src="chrome://resources/js/util.js"></script>
  <script src="strings.js"></script>
  <script src="hello_world.js"></script>
</head>
<body i18n-values=".style.fontFamily:fontfamily;.style.fontSize:fontsize">
  <h1 i18n-content="helloWorldTitle"></h1>
  <p id="welcome-message"></p>
 <script src="chrome://resources/js/i18n_template2.js"></script>
</body>
</html>
{%endhighlight%}

添加css文件
{%highlight css%}
src/chrome/browser/resources/hello_world.css

p {
  white-space: pre-wrap;
}
{%endhighlight%}

添加js文件
{%highlight js%}
src/chrome/browser/resources/hello_world.js

cr.define('hello_world', function() {
  'use strict';

  /**
   * Be polite and insert translated hello world strings for the user on loading.
   */
  function initialize() {
    $('welcome-message').textContent = loadTimeData.getStringF('welcomeMessage',
    loadTimeData.getString('userName'));
  }

  // Return an object with all of the exports.
  return {
    initialize: initialize,
  };
});

document.addEventListener('DOMContentLoaded', hello_world.initialize);
{%endhighlight%}
#### 2 把资源文件添加到Chrome

使用src/chrome/browser/browser_resources.grd文件来添加资源文件。

src/chrome/browser/browser_resources.grd文件里面添加hello_world的html、css、js文件
{%highlight xml%}
<include name="IDR_HELLO_WORLD_HTML" file="resources\hello_world.html" type="BINDATA" />
<include name="IDR_HELLO_WORLD_CSS" file="resources\hello_world.css" type="BINDATA" />
<include name="IDR_HELLO_WORLD_JS" file="resources\hello_world.js" type="BINDATA" />
{%endhighlight%}
#### 3 为新的chrome URL添加URL标识

URL标识被存放在src/chrome/common/url_constants.*，在这里添加指向新资源的URL标识
{%highlight c++%}
src/chrome/common/url_constants.h

extern const char kChromeUIHelloWorldURL[];
extern const char kChromeUIHelloWorldHost[];

src/chrome/common/url_constants.cc

const char kChromeUIHelloWorldURL[] = "chrome://hello-world/";
const char kChromeUIHelloWorldHost[] = "hello-world";
{%endhighlight%}
#### 4 添加本地化字符串

在html页面上显示的一些字符串，建议按照chrome的标准，进行本地化，这样方便输出多语言版本，存放字符串的文件为src/chrome/app/generated_resources.grd，这个文件内对应的是英文字符串，多语言文件存放在src/chrome/app/resources/generated_resources_*.xtb，例如中文的字符串对应文是generated_resources_zh-CN.xtb。
    generated_resources.grd里面的字符串首位的空格都会忽略，添加的时候如果可以加一些特殊字符来区分，例如’&lt;’、’&gt;’、 ‘&amp;’、 ‘&quot;’、 ‘&apos;’。其中translation id必须唯一，如果多个相同会编译错误。
{%highlight xml%}
src/chrome/app/generated_resources.grd

<message name="IDS_HELLO_WORLD_TITLE" desc="A happy message saying hello to the world">
   Hello World!
</message>
<message name="IDS_HELLO_WORLD_WELCOME_TEXT" desc="Message welcoming the user to the hello world page">
	Welcome to this fancy Hello World page <ph name="WELCOME_NAME">$1<ex>Chromium User</ex></ph>!
</message>

src/chrome/app/resources/generated_resources_zh-CN.xtb

<translation id=“7856356408326784566">你好 世界！</translation>
<translation id=“1952586428076772168">欢迎来到这个奇妙的Hello World页面 <ph name="WELCOME_NAME" />！</translation>
{%endhighlight%}
#### 5 添加处理chrome://hello-world/请求的WebUI类

接下来我们需要一个类来处理新资源URL的请求，通常这个类是继承自ChromeWebUI（WebUI对话框则是继承自HtmlDialogUI）
{%highlight c++%}
src/chrome/browser/ui/webui/hello_world_ui.h

#ifndef CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_UI_H_
#define CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_UI_H_
#pragma once

#include "content/public/browser/web_ui_controller.h"

// The WebUI for chrome://hello-world
class HelloWorldUI : public content::WebUIController {
public:
  explicit HelloWorldUI(content::WebUI* web_ui);
  virtual ~HelloWorldUI();
 private: 
  DISALLOW_COPY_AND_ASSIGN(HelloWorldUI);
};

#endif  // CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_UI_H_

src/chrome/browser/ui/webui/hello_world_ui.cc

#include "chrome/browser/ui/webui/hello_world_ui.h"

#include "chrome/browser/profiles/profile.h"
#include "chrome/common/url_constants.h"
#include "content/public/browser/web_ui_data_source.h"
#include "grit/browser_resources.h"
#include "grit/generated_resources.h"

HelloWorldUI::HelloWorldUI(content::WebUI* web_ui)
    : content::WebUIController(web_ui) {
  // Set up the chrome://hello-world source.
  content::WebUIDataSource* html_source =
	content::WebUIDataSource::Create(chrome::kChromeUIHelloWorldHost);
  html_source->SetUseJsonJSFormatV2();

 // Localized strings.
 html_source->AddLocalizedString("helloWorldTitle", IDS_HELLO_WORLD_TITLE);
 html_source->AddLocalizedString("welcomeMessage", IDS_HELLO_WORLD_WELCOME_TEXT);

 // As a demonstration of passing a variable for JS to use we pass in the name "Bob".
 html_source->AddString("userName", "Bob");
 html_source->SetJsonPath("strings.js");

 // Add required resources.
 html_source->AddResourcePath("hello_world.css", IDR_HELLO_WORLD_CSS);
 html_source->AddResourcePath("hello_world.js", IDR_HELLO_WORLD_JS);
 html_source->SetDefaultResource(IDR_HELLO_WORLD_HTML);

 Profile* profile = Profile::FromWebUI(web_ui);
 content::WebUIDataSource::Add(profile, html_source);
}

HelloWorldUI::~HelloWorldUI() {
}
{%endhighlight%}
#### 6 添加新资源到Chrome

将新添加的类添加到Chrome里面，如果添加新的类的话，需要在src/chrome/chrome_browser_ui.gypi文件中添加，这样可以在项目编译的时候链接这些文件。
{%highlight xml%}
src/chrome/chrome_browser_ui.gypi

'sources': [
...
'browser/ui/webui/hello_world_ui.cc',
'browser/ui/webui/hello_world_ui.h',
{%endhighlight%}

#### 7 添加WebUI URL的解析

Chrome WebUI工厂方法类里面添加处理新请求的代码
{%highlight c++%}
src/chrome/browser/ui/webui/chrome_web_ui_controller_factory.cc

#include "chrome/browser/ui/webui/hello_world_ui.h"
...
if (url.host() == chrome::kChromeUIHelloWorldHost)
  return &NewWebUI<HelloWorldUI>;
{%endhighlight%}
#### 9 添加与js的回调

如果我们想通过js调用cC++去执行代码时，可以在对应的类里面添加
{%highlight c++%}
src/chrome/browser/ui/webui/hello_world_ui.h

#include "chrome/browser/ui/webui/chrome_web_ui.h"
+
+ namespace base {
+   class ListValue;
+ }  // namespace base

// The WebUI for chrome://hello-world
...
// Set up the chrome://hello-world source.
ChromeWebUIDataSource* html_source =
    new ChromeWebUIDataSource(chrome::kChromeUIHelloWorldHost);
+
+   // Register callback handler.
+   RegisterMessageCallback("addNumbers",
+       base::Bind(&HelloWorldUI::AddNumbers,
+                  base::Unretained(this)));

// Localized strings.
...
virtual ~HelloWorldUI();
+
+  private:
+   // Add two numbers together using integer arithmetic.
+   void AddNumbers(const base::ListValue* args);
	DISALLOW_COPY_AND_ASSIGN(HelloWorldUI);
};

src/chrome/browser/ui/webui/hello_world_ui.cc

  #include "chrome/browser/ui/webui/hello_world_ui.h"
+
+ #include "base/values.h"
  #include "chrome/browser/profiles/profile.h"
...
HelloWorldUI::~HelloWorldUI() {
  }
+
+ void HelloWorldUI::AddNumbers(const base::ListValue* args) {
+   int term1, term2;
+   if (!args->GetInteger(0, &term1) || !args->GetInteger(1, &term2))
+     return;
+   base::FundamentalValue result(term1 + term2);
+   CallJavascriptFunction("hello_world.addResult", result);
+ }


src/chrome/browser/resources/hello_world.js

function initialize() {
+     chrome.send('addNumbers', [2, 2]);
}
+
+   function addResult(result) {
+     alert('The result of our C++ arithmetic: 2 + 2 = ' + result);
+   }

return {
+     addResult: addResult,
	initialize: initialize,
};
{%endhighlight%}
这个调用是异步的，必须等待C++那边去调用js函数来得到结果


### 创建WebUI对话框

创建WebUI对话框只与上面的有两个地方不同，处理请求的类必须继承自HtmlDialogUI类，创建一个HtmlDialogUIDelegate类来负责运行对话框。

#### 1 修改继承的父类
{%highlight c++%}
src/chrome/browser/ui/webui/hello_world_ui.h

- #include "#chrome/browser/ui/webui/chrome_web_ui.h"
+ #include "#chrome/browser/ui/webui/html_dialog_ui.h"

- class HelloWorldUI : public ChromeWebUI {
+ class HelloWorldUI : public HtmlDialogUI {
{%endhighlight%}
#### 2 创建HtmlDialogUIDelegate类来实例化对话框
{%highlight c++%}
src/chrome/browser/ui/webui/hello_world.h

#ifndef CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_H_
#define CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_H_
#pragma once

#include "chrome/browser/ui/webui/html_dialog_ui.h"

class HelloWorldDialog : private HtmlDialogUIDelegate {
public:
  // Shows the Hello World dialog.
  static void ShowDialog();
  virtual ~HelloWorldDialog();

private:
  // Construct a Hello World dialog
  explicit HelloWorldDialog();

  // Overridden from HtmlDialogUI::Delegate:
  virtual bool IsDialogModal() const OVERRIDE;
  virtual string16 GetDialogTitle() const OVERRIDE;
  virtual GURL GetDialogContentURL() const OVERRIDE;
  virtual void GetWebUIMessageHandlers(
	std::vector<WebUIMessageHandler*>* handlers) const OVERRIDE;
  virtual void GetDialogSize(gfx::Size* size) const OVERRIDE;
  virtual std::string GetDialogArgs() const OVERRIDE;
  virtual void OnDialogClosed(const std::string& json_retval) OVERRIDE;
  virtual void OnCloseContents(
	TabContents* source, bool* out_close_dialog) OVERRIDE;
  virtual bool ShouldShowDialogTitle() const OVERRIDE;

  DISALLOW_COPY_AND_ASSIGN(HelloWorldDialog);
};

#endif  // CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_H_

src/chrome/browser/ui/webui/hello_world.cc

#include "base/utf_string_conversions.h"
#include "chrome/browser/ui/browser.h"
#include "chrome/browser/ui/browser_list.h"
#include "chrome/browser/ui/webui/hello_world.h"
#include "chrome/common/url_constants.h"

void HelloWorldDialog::ShowDialog() {
  Browser* browser = BrowserList::GetLastActive();
  DCHECK(browser);
  browser->BrowserShowHtmlDialog(new HelloWorldDialog(), NULL);
}

HelloWorldDialog::HelloWorldDialog() {
}

HelloWorldDialog::~HelloWorldDialog() {
}

bool HelloWorldDialog::IsDialogModal() {
  return false;
}

string16 HelloWorldDialog::GetDialogTitle() {
  return UTF8ToUTF16("Hello World");
}

GURL HelloWorldDialog::GetDialogContentURL() const {
  return GURL(chrome::kChromeUIHelloWorldURL);
}

void HelloWorldDialog::GetWebUIMessageHandlers(
  std::vector<WebUIMessageHandler*>* handlers) const {
}

void HelloWorldDialog::GetDialogSize(gfx::Size* size) const {
  size->SetSize(600, 400);
}

std::string HelloWorldDialog::GetDialogArgs() const {
  return std::string();
}

void HelloWorldDialog::OnDialogClosed(const std::string& json_retval) {
  delete this;
}

void HelloWorldDialog::OnCloseContents(TabContents* source,
    bool* out_close_dialog) {
  if (out_close_dialog)
    *out_close_dialog = true;
}

bool HelloWorldDialog::ShouldShowDialogTitle() const {
  return true;
}
{%endhighlight%}
你可以用HelloWorldDialog::ShowDialog()来调用这个新的对话框。

#### 3 传递参数给WebUI对话框

通过HtmlDialogUIDelegate::GetDialogArgs()函数来传递参数给对话框。
{%highlight c++%}
src/chrome/browser/ui/webui/hello_world.h

-   static void ShowDialog();
+   static void ShowDialog(std::string message);

+   // The message to be displayed to the user.
+   std::string message_;
+
    DISALLOW_COPY_AND_ASSIGN(HelloWorldDialog);
  };


src/chrome/browser/ui/webui/hello_world.cc

- HelloWorldDialog::HelloWorldDialog() {
+ HelloWorldDialog::HelloWorldDialog(std::string message)
+     : message_(message) {
  }

  std::string HelloWorldDialog::GetDialogArgs() const {
-   return std::string();
+   return message_;
  }

src/chrome/browser/resources/hello_world.js:

function initialize() {
+     document.getElementsByTagName('p')[0].textContent = chrome.dialogArguments;
}
{%endhighlight%}