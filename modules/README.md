# MagicMirror² 모듈 개발 문서

이 문서에는 당신만의 MagicMirror² 모듈을 만드는 방법에 대해서 설명되어 있습니다.

## 모듈의 구성

모든 모듈은 `modules` 폴더에 저장되어있으며, 기본 모듈은 `modules/default` 폴더에 함께 저장되어있습니다. 또한, 당신이 만든 모듈은 `modules` 폴더 아래의 또 다른 하위 폴더에 저장되어야합니다.
당신이 `modules` 폴더 속에 만든 모든 파일과 폴더는 새로운 버전의 MagicMirror²를 받을 때 손상될 수 없게, git으로부터 무시된다는 것을 알아두시길 바랍니다.

하나의 폴더에는 하나의, 혹은 그보다 많은 양의 모듈이 존재할 수 있습니다. 단, 각 모듈의 이름은 중복될 수 없습니다.  비슷한(혹은 같은) 이름의 모듈이 서로 다른 폴더에 존재한다고 하더라도, 해당 모듈을 동시에 로딩할 수 없다는 것을 명심하십시오.

### 파일 구성
- **modulename/modulename.js** - 당신이 만든 모듈의 core 모듈 스크립트입니다.
- **modulename/node_helper.js** - node 스크립트에 의하여 불려질 helper 스크립트입니다. helper 스크립트와 core 스크립트는 별도로 통합된 소켓 시스템에 의해 서로에게 정보를 전달할 수 있습니다. 해당 파일은 필수가 아닙니다(optional).
- **modulename/public** - 해당 폴더에 있는 모든 파일은 브라우저 내에서 `/modulename/filename.ext`과 같이 접근 가능합니다.
- **modulename/anyfileorfolder** - 해당 폴더에 있는 모든 파일 및 폴더는 core 모듈 스크립트에서 접근 가능합니다. 예컨대, *modulename/css/modulename.css*는 추가적인 모듈 스타일을 관리하기 위한 좋은 경로가 될 수 있습니다.

## Core 모듈 파일: modulename.js
모듈의 내용이 정의되는 필수 스크립트 파일입니다. 가장 간단하게, 다음과 같이 작성될 수 있습니다:
````javascript
Module.register("modulename",{});
````

물론 위의 모듈은 아무런 추가적인 작업을 할 수 없기 때문에, 간단한 모듈 중 하나인 **helloworld**의 core 모듈 스크립트를 살펴보며 기본적인 설명을 덧붙이도록 하겠습니다:
````javascript
//helloworld.js:

Module.register("helloworld",{
	// Default module config.
	defaults: {
		text: "Hello World!"
	},

	// Override dom generator.
	getDom: function() {
		var wrapper = document.createElement("div");
		wrapper.innerHTML = this.config.text;
		return wrapper;
	}
});
````

위 스크립트에서 `Module.register()`는 두 개의 인수를 받고 있습니다: 모듈의 이름과 모듈의 세부적인 설정에 관한 내용이 그것입니다.

### 참조 가능한 모듈 객체의 속성 값
모듈이 초기화되고 나면, 기본적인 모듈 객체의 속성의 값을 참조할 수 있습니다.

####`this.name`
**String**

모듈의 이름을 나타냅니다.

####`this.identifier`
**String**

해당 모듈 객체의 고유번호(unique identifier)를 나타냅니다.

####`this.hidden`
**Boolean**

해당 모듈의 모습이 숨겨졌는지(사라졌는지)에 대한 Boolean값을 나타냅니다.

####`this.config`
**Boolean**

사용자의 config.js 파일에 작성해둔 모듈의 속성 값입니다.
사용자의 config을 통한 추가적인 설정이 없었다면, 모듈이 가질 수 있는 기본 속성을 갖습니다.

####`this.data`
**Object**

모듈 객체에 대한 metadata를 포함하고 있는 data 객체입니다.
- `data.classes` - 모듈 dom wrapper에 추가된 classes의 정보입니다.
- `data.file` - core 모듈 파일의 이름입니다.
- `data.path` - 해당 모듈이 존재하는 폴더의 경로입니다.
- `data.header` - 모듈에 추가된 header의 정보입니다.
- `data.position` - 모듈 객체가 표시될 위치의 정보입니다.

####`defaults: {}`
당신이 만든 모듈의 기본 속성을 작성할 수 있는 객체입니다. 해당 객체에 작성된 속성 값은 사용자의 config.js의 속성 값과 함께 사용될 것입니다. 모든 모듈 속성 값은 `this.config.propertyName`와 같이 참조될 수 있습니다. 이후 해당 부분에 대해 더 자세히 알려드리도록 하겠습니다.

####'requiresVersion:'

*2.1.0 버전에 추가된 내용입니다.*

모듈을 구동시키기에 필요한 MagicMirror 프레임워크의 최소 버전입니다. 만약 따로 정의되어있지 않다면, 시스템이 자동적으로 모듈의 현재 버전과 MagicMirror의 현재 버전을 비교할 것입니다. 만약 모듈의 현재 버전의 업데이트가 필요하다고 판단되면, MagicMirror는 모듈을 구동하지 않을 것입니다. Node helper 스크립트에서 해당 값을 확실하게 작성하시길 바랍니다.

**주의:** 해당 속성은 MagicMirror 버전 2.1.0에서 소개되었기 때문에, 그보다 오래된 버전에서는 작동하지 않을 것입니다. issue를 작성하기 전에 꼭 참고하시길 바랍니다.

예시:
````javascript
requiresVersion: "2.1.0",
````

### 내부적으로 사용가능한 모듈 함수

####`init()`
This method is called when a module gets instantiated. In most cases you do not need to subclass this method.

####`start()`
This method is called when all modules are loaded an the system is ready to boot up. Keep in mind that the dom object for the module is not yet created. The start method is a perfect place to define any additional module properties:

**Example:**
````javascript
start: function() {
	this.mySpecialProperty = "So much wow!";
	Log.log(this.name + ' is started!');
}
````

####`getScripts()`
**Should return: Array**

The getScripts method is called to request any additional scripts that need to be loaded. This method should therefore return an array with strings. If you want to return a full path to a file in the module folder, use the `this.file('filename.js')` method. In all cases the loader will only load a file once. It even checks if the file is available in the default vendor folder.

**Example:**
````javascript
getScripts: function() {
	return [
		'script.js', // will try to load it from the vendor folder, otherwise it will load is from the module folder.
		'moment.js', // this file is available in the vendor folder, so it doesn't need to be available in the module folder.
		this.file('anotherfile.js'), // this file will be loaded straight from the module folder.
		'https://code.jquery.com/jquery-2.2.3.min.js',  // this file will be loaded from the jquery servers.
	]
}

````
**Note:** If a file can not be loaded, the boot up of the mirror will stall. Therefore it's advised not to use any external urls.


####`getStyles()`
**Should return: Array**

The getStyles method is called to request any additional stylesheets that need to be loaded. This method should therefore return an array with strings. If you want to return a full path to a file in the module folder, use the `this.file('filename.css')` method. In all cases the loader will only load a file once. It even checks if the file is available in the default vendor folder.

**Example:**
````javascript
getStyles: function() {
	return [
		'script.css', // will try to load it from the vendor folder, otherwise it will load is from the module folder.
		'font-awesome.css', // this file is available in the vendor folder, so it doesn't need to be avialable in the module folder.
		this.file('anotherfile.css'), // this file will be loaded straight from the module folder.
		'https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css',  // this file will be loaded from the bootstrapcdn servers.
	]
}

````
**Note:** If a file can not be loaded, the boot up of the mirror will stall. Therefore it's advised not to use any external urls.

####`getTranslations()`
**Should return: Dictionary**

The getTranslations method is called to request translation files that need to be loaded. This method should therefore return a dictionary with the files to load, identified by the country's short name.

If the module does not have any module specific translations, the function can just be omitted or return `false`.

**Example:**
````javascript
getTranslations: function() {
	return {
			en: "translations/en.json",
			de: "translations/de.json"
	}
}

````

####`getDom()`
**Should return:** Dom Object

Whenever the MagicMirror needs to update the information on screen (because it starts, or because your module asked a refresh using `this.updateDom()`), the system calls the getDom method. This method should therefore return a dom object.

**Example:**
````javascript
getDom: function() {
	var wrapper = document.createElement("div");
	wrapper.innerHTML = 'Hello world!';
	return wrapper;
}

````

####`getHeader()`
**Should return:** String

Whenever the MagicMirror needs to update the information on screen (because it starts, or because your module asked a refresh using `this.updateDom()`), the system calls the getHeader method to retrieve the module's header. This method should therefor return a string. If this method is not subclassed, this function will return the user's configured header.

If you want to use the original user's configured header, reference `this.data.header`.

**NOTE:** If the user did not configure a default header, no header will be displayed and thus this method will not be called.

**Example:**
````javascript
getHeader: function() {
	return this.data.header + ' Foo Bar';
}

````

####`notificationReceived(notification, payload, sender)`

That MagicMirror core has the ability to send notifications to modules. Or even better: the modules have the possibility to send notifications to other modules. When this module is called, it has 3 arguments:

- `notification` - String - The notification identifier.
- `payload` - AnyType - The payload of a notification.
- `sender` - Module - The sender of the notification. If this argument is `undefined`, the sender of the notififiction is the core system.

**Example:**
````javascript
notificationReceived: function(notification, payload, sender) {
	if (sender) {
		Log.log(this.name + " received a module notification: " + notification + " from sender: " + sender.name);
	} else {
		Log.log(this.name + " received a system notification: " + notification);
	}
}
````

**Note:** the system sends two notifications when starting up. These notifications could come in handy!


- `ALL_MODULES_STARTED` - All modules are started. You can now send notifications to other modules.
- `DOM_OBJECTS_CREATED` - All dom objects are created. The system is now ready to perform visual changes.


####`socketNotificationReceived: function(notification, payload)`
When using a node_helper, the node helper can send your module notifications. When this module is called, it has 2 arguments:

- `notification` - String - The notification identifier.
- `payload` - AnyType - The payload of a notification.

**Note 1:** When a node helper sends a notification, all modules of that module type receive the same notifications. <br>
**Note 2:** The socket connection is established as soon as the module sends its first message using [sendSocketNotification](thissendsocketnotificationnotification-payload).

**Example:**
````javascript
socketNotificationReceived: function(notification, payload) {
	Log.log(this.name + " received a socket notification: " + notification + " - Payload: " + payload);
},
````

####`suspend()`
When a module is hidden (using the `module.hide()` method), the `suspend()` method will be called. By subclassing this method you can perform tasks like halting the update timers.

####`resume()`
When a module will be shown after it was previously hidden (using the `module.show()` method), the `resume()` method will be called. By subclassing this method you can perform tasks restarting the update timers.


### Module instance methods

Each module instance has some handy methods which can be helpful building your module.


####`this.file(filename)`
***filename* String** - The name of the file you want to create the path for.<br>
**Returns String**

If you want to create a path to a file in your module folder, use the `file()` method. It returns the path to the filename given as the attribute. Is method comes in handy when configuring the [getScripts](#getscripts) and [getStyles](#getstyles) methods.

####`this.updateDom(speed)`
***speed* Number** - Optional. Animation speed in milliseconds.<br>

Whenever your module need to be updated, call the `updateDom(speed)` method. It requests the MagicMirror core to update its dom object. If you define the speed, the content update will be animated, but only if the content will really change.

As an example: the clock modules calls this method every second:

````javascript
...
start: function() {
	var self = this;
	setInterval(function() {
		self.updateDom(); // no speed defined, so it updates instantly.
	}, 1000); //perform every 1000 milliseconds.
},
...
````

####`this.sendNotification(notification, payload)`
***notification* String** - The notification identifier.<br>
***payload* AnyType** - Optional. A notification payload.<br>

If you want to send a notification to all other modules, use the `sendNotification(notification, payload)`. All other modules will receive the message via the [notificationReceived](#notificationreceivednotification-payload-sender) method. In that case, the sender is automatically set to the instance calling the sendNotification method.

**Example:**
````javascript
this.sendNotification('MYMODULE_READY_FOR_ACTION', {foo:bar});
````

####`this.sendSocketNotification(notification, payload)`
***notification* String** - The notification identifier.<br>
***payload* AnyType** - Optional. A notification payload.<br>

If you want to send a notification to the node_helper, use the `sendSocketNotification(notification, payload)`. Only the node_helper of this module will receive the socket notification.

**Example:**
````javascript
this.sendSocketNotification('SET_CONFIG', this.config);
````

####`this.hide(speed, callback, options)`
***speed* Number** - Optional (Required when setting callback or options), The speed of the hide animation in milliseconds.
***callback* Function** - Optional, The callback after the hide animation is finished.
***options* Function** - Optional, Object with additional options for the hide action (see below). (*Introduced in version: 2.1.0.*)

To hide a module, you can call the `hide(speed, callback)` method. You can call the hide method on the module instance itself using `this.hide()`, but of course you can also hide another module using `anOtherModule.hide()`.

Possible configurable options:

- `lockString` - String - When setting lock string, the module can not be shown without passing the correct lockstring. This way (multiple) modules can prevent a module from showing. It's considered best practice to use your modules identifier as the locksString: `this.identifier`. See *visibility locking* below.


**Note 1:** If the hide animation is canceled, for instance because the show method is called before the hide animation was finished, the callback will not be called.<br>
**Note 2:** If the hide animation is hijacked (an other method calls hide on the same module), the callback will not be called.<br>
**Note 3:** If the dom is not yet created, the hide method won't work. Wait for the `DOM_OBJECTS_CREATED` [notification](#notificationreceivednotification-payload-sender).


####`this.show(speed, callback, options)`
***speed* Number** - Optional (Required when setting callback or options), The speed of the show animation in milliseconds.
***callback* Function** - Optional, The callback after the show animation is finished.
***options* Function** - Optional, Object with additional options for the show action (see below). (*Introduced in version: 2.1.0.*)

To show a module, you can call the `show(speed, callback)` method. You can call the show method on the module instance itself using `this.show()`, but of course you can also show another module using `anOtherModule.show()`.

Possible configurable options:

- `lockString` - String - When setting lock string, the module can not be shown without passing the correct lockstring. This way (multiple) modules can prevent a module from showing. See *visibility locking* below.
- `force` - Boolean - When setting the force tag to `true`, the locking mechanism will be overwritten. Use this option with caution. It's considered best practice to let the usage of the force option be use- configurable. See *visibility locking* below.

**Note 1:** If the show animation is canceled, for instance because the hide method is called before the show animation was finished, the callback will not be called.<br>
**Note 2:** If the show animation is hijacked (an other method calls show on the same module), the callback will not be called.<br>
**Note 3:** If the dom is not yet created, the show method won't work. Wait for the `DOM_OBJECTS_CREATED` [notification](#notificationreceivednotification-payload-sender).

####Visibility locking

(*Introduced in version: 2.1.0.*)

Visiblity locking helps the module system to prevent unwanted hide/show actions. The following scenario explains the concept:

**Module B asks module A to hide:**
````javascript
moduleA.hide(0, {lockString: "module_b_identifier"});
````
Module A is now hidden, and has an lock array with the following strings:
````javascript
moduleA.lockStrings == ["module_b_identifier"]
moduleA.hidden == true
````
**Module C asks module A to hide:**
````javascript
moduleA.hide(0, {lockString: "module_c_identifier"});
````
Module A is now hidden, and has an lock array with the following strings:
````javascript
moduleA.lockStrings == ["module_b_identifier", "module_c_identifier"]
moduleA.hidden == true
````
**Module B asks module A to show:**
````javascript
moduleA.show(0, {lockString: "module_b_identifier"});
````
The lockString will be removed from moduleA’s locks array, but since there still is an other lock string available, the module remains hidden:
````javascript
moduleA.lockStrings == ["module_c_identifier"]
moduleA.hidden == true
````
**Module C asks module A to show:**
````javascript
moduleA.show(0, {lockString: "module_c_identifier"});
````
The lockString will be removed from moduleA’s locks array, and since this will result in an empty lock array, the module will be visible:
````javascript
moduleA.lockStrings == []
moduleA.hidden == false
````

**Note:** The locking mechanism can be overwritten by using the force tag:
````javascript
moduleA.show(0, {force: true});
````
This will reset the lockstring array, and will show the module.
````javascript
moduleA.lockStrings == []
moduleA.hidden == false
````

Use this `force` method with caution. See `show()` method for more information.



####`this.translate(identifier)`
***identifier* String** - Identifier of the string that should be translated.

The Magic Mirror contains a convenience wrapper for `l18n`. You can use this to automatically serve different translations for your modules based on the user's `language` configuration.

If no translation is found, a fallback will be used. The fallback sequence is as follows:
- 1. Translation as defined in module translation file of the user's preferred language.
- 2. Translation as defined in core translation file of the user's preferred language.
- 3. Translation as defined in module translation file of the fallback language (the first defined module translation file).
- 4. Translation as defined in core translation file of the fallback language (the first defined core translation file).
- 5. The key (identifier) of the translation.

When adding translations to your module, it's a good idea to see if an apropriate translation is already available in the [core translation files](https://github.com/MichMich/MagicMirror/tree/master/translations). This way, your module can benefit from the existing translations.

**Example:**
````javascript
this.translate("INFO") //Will return a translated string for the identifier INFO
````

**Example json file:**
````javascript
{
  "INFO": "Really important information!"
}
````

**Note:** although comments are officially not supported in JSON files, MagicMirror allows it by stripping the comments before parsing the JSON file. Comments in translation files could help other translators.


## The Node Helper: node_helper.js

The node helper is a Node.js script that is able to do some backend task to support your module. For every module type, only one node helper instance will be created. For example: if your MagicMirror uses two calendar modules, there will be only one calendar node helper instantiated.

**Note:** Because there is only one node helper per module type, there is no default config available within your module. It's your task to send the desired config from your module to your node helper.

In it's most simple form, the node_helper.js file must contain:

````javascript
var NodeHelper = require("node_helper");
module.exports = NodeHelper.create({});
````

Of course, the above helper would not do anything useful. So with the information above, you should be able to make it a bit more sophisticated.

### Available module instance properties

####`this.name`
**String**

The name of the module

####`this.path`
**String**

The path of the module

####`this.expressApp`
**Express App Instance**

This is a link to the express instance. It will allow you to define extra routes.

**Example:**
````javascript
start: function() {
	this.expressApp.get('/foobar', function (req, res) {
		res.send('GET request to /foobar');
	});
}
````

**Note:** By default, a public path to your module's public folder will be created:
````javascript
this.expressApp.use("/" + this.name, express.static(this.path + "/public"));
````

####`this.io`
**Socket IO Instance**

This is a link to the IO instance. It will allow you to do some Socket.IO magic. In most cases you won't need this, since the Node Helper has a few convenience methods to make this simple.


####'requiresVersion:'
*Introduced in version: 2.1.0.*

A string that defines the minimum version of the MagicMirror framework. If it is set, the system compares the required version with the users version. If the version of the user is out of date, it won't run the module.

**Note:** Since this check is introduced in version 2.1.0, this check will not be run in older versions. Keep this in mind if you get issue reports on your module.

Example:
````javascript
requiresVersion: "2.1.0",
````

### Subclassable module methods

####`init()`
This method is called when a node helper gets instantiated. In most cases you do not need to subclass this method.

####`start()`
This method is called when all node helpers are loaded and the system is ready to boot up. The start method is a perfect place to define any additional module properties:

**Example:**
````javascript
start: function() {
	this.mySpecialProperty = "So much wow!";
	Log.log(this.name + ' is started!');
}
````

####`socketNotificationReceived: function(notification, payload)`
With this method, your node helper can receive notifications from your modules. When this method is called, it has 2 arguments:

- `notification` - String - The notification identifier.
- `payload` - AnyType - The payload of a notification.

**Note:** The socket connection is established as soon as the module sends its first message using [sendSocketNotification](thissendsocketnotificationnotification-payload).

**Example:**
````javascript
socketNotificationReceived: function(notification, payload) {
	Log.log(this.name + " received a socket notification: " + notification + " - Payload: " + payload);
},
````

### Module instance methods

Each node helper has some handy methods which can be helpful building your module.

####`this.sendSocketNotification(notification, payload)`
***notification* String** - The notification identifier.<br>
***payload* AnyType** - Optional. A notification payload.<br>

If you want to send a notification to all your modules, use the `sendSocketNotification(notification, payload)`. Only the module of your module type will receive the socket notification.

**Note:** Since all instances of your module will receive the notifications, it's your task to make sure the right module responds to your messages.

**Example:**
````javascript
this.sendSocketNotification('SET_CONFIG', this.config);
````

## MagicMirror Helper Methods

The core Magic Mirror object: `MM` has some handy method that will help you in controlling your and other modules. Most of the `MM` methods are available via convenience methods on the Module instance.

### Module selection
The only additional method available for your module, is the feature to retrieve references to other modules. This can be used to hide and show other modules.

####`MM.getModules()`
**Returns Array** - An array with module instances.<br>

To make a selection of all currently loaded module instances, run the `MM.getModules()` method. It will return an array with all currently loaded module instances. The returned array has a lot of filtering methods. See below for more info.

**Note:** This method returns an empty array if not all modules are started yet. Wait for the `ALL_MODULES_STARTED` [notification](#notificationreceivednotification-payload-sender).


#####`.withClass(classnames)`
***classnames* String or Array** - The class names on which you want to filter.
**Returns Array** - An array with module instances.<br>

If you want to make a selection based on one or more class names, use the withClass method on a result of the `MM.getModules()` method. The argument of the `withClass(classname)` method can be an array, or space separated string.

**Examples:**
````javascript
var modules = MM.getModules().withClass('classname');
var modules = MM.getModules().withClass('classname1 classname2');
var modules = MM.getModules().withClass(['classname1','classname2']);
````

#####`.exceptWithClass(classnames)`
***classnames* String or Array** - The class names of the modules you want to remove from the results.
**Returns Array** - An array with module instances.<br>

If you to remove some modules from a selection based on a classname, use the exceptWithClass method on a result of the `MM.getModules()` method. The argument of the `exceptWithClass(classname)` method can be an array, or space separated string.

**Examples:**
````javascript
var modules = MM.getModules().exceptWithClass('classname');
var modules = MM.getModules().exceptWithClass('classname1 classname2');
var modules = MM.getModules().exceptWithClass(['classname1','classname2']);
````

#####`.exceptModule(module)`
***module* Module Object** - The reference to a module you want to remove from the results.
**Returns Array** - An array with module instances.<br>

If you to remove a specific module instance from a selection based on a classname, use the exceptWithClass method on a result of the `MM.getModules()` method. This can be helpful if you want to select all module instances except the instance of your module.

**Examples:**
````javascript
var modules = MM.getModules().exceptModule(this);
````

Of course, you can combine all of the above filters:

**Example:**
````javascript
var modules = MM.getModules().withClass('classname1').exceptwithClass('classname2').exceptModule(aModule);
````

#####`.enumerate(callback)`
***callback* Function(module)** - The callback run on every instance.

If you want to perform an action on all selected modules, you can use the `enumerate` function:

````javascript
MM.getModules().enumerate(function(module) {
    Log.log(module.name);
});
````

**Example:**
To hide all modules except the your module instance, you could write something like:
````javascript
Module.register("modulename",{
	//...
	notificationReceived: function(notification, payload, sender) {
		if (notification === 'DOM_OBJECTS_CREATED') {
			MM.getModules().exceptModule(this).enumerate(function(module) {
				module.hide(1000, function() {
					//Module hidden.
				});
			});
		}
	},
	//...
});
````

## MagicMirror Logger

The Magic Mirror contains a convenience wrapper for logging. Currently, this logger is a simple proxy to the original `console.log` methods. But it might get additional features in the future. The Loggers is currently only available in the core module file (not in the node_helper).

**Examples:**
````javascript
Log.info('error');
Log.log('log');
Log.error('info');
````
