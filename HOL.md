#Compatibility, Developer Tools and Performance#

## Overview ##

Developers building modern HTML5 applications have to contend with ever-expanding standards, increasing application complexity, a much wider array of target browsers and platforms, and more sophisticated user expectations and requirements. In fact, the HTML5 and CSS3 standards are each composed of multiple specifications documents because they have both grown so much. Managing all this variety is not easy, especially when different browsers behave differently or interpret the standards differently.

Internet Explorer 10 represents a significant milestone in Internet Explorer’s evolution, especially in helping developers deal with all this complexity. Embracing the idea that HTML5 applications are becoming more and more prevalent, whether inside or outside the browser, Microsoft has put a lot of effort into making Internet Explorer more standards-compliant than ever, improving the Internet Explorer Developer Tools, and boosting the browser’s performance. This lab focuses on some of the tooling and feature improvements introduced in Internet Explorer 10.

### Objectives ###

In this hands-on lab, you will learn how to:

-	Pass data and instructions from a web worker to the main script

-	Debug web workers using Internet Explorer 10’s integrated debugger

-	Ensure cross-browser and backwards compatibility using the Compat Inspector

 
### Prerequisites ###

- Internet Explorer 10
- An HTML & JavaScript editor of your choice
- Prior knowledge of JavaScript

## Exercises ##

This hands-on lab includes the following exercises:

1. [Exercise 1: Debugging Web Workers](#Exercise1)

1. [Exercise 2: Using the Compat Inspector](#Exercise2)

Estimated time to complete this lab: **20-30 minutes**.

<a name="Exercise1" />
### Exercise 1: Debugging Web Workers ###

JavaScript supports asynchronous behavior, which is what makes it possible to handle events and use Ajax to communicate with a server. However, asynchrony can only go so far. Problems begin to arise when the asynchronous operations perform heavy computations. Image filters such as those in the **TheFacePlace** application, for example, are algorithms that are applied to every pixel in the image. These algorithms are CPU-intensive and can be time-consuming. Since the operations are performed on the same thread as the browser window, they can block the UI and make it unresponsive.

HTML5 **Web Workers** come to the rescue. Web Workers are scripts that run in the background and do not interact with the UI thread. They do not directly affect anything that happens on the page or in the DOM. This makes them very useful for heavy computations, such as those required by image filters. This also helps improve page performance by offloading time-consuming operations.

However, because they do not interact with the UI, they can be very difficult to debug. They have no access to the **window** object, so they cannot write to the **console** or open **alert** dialogs. Since they run in a separate environment, the debugger cannot simply step through from one script to another. Internet Explorer 10 overcomes this limitation and lets you debug your worker scripts inside the browser. You can place breakpoints and analyze the stack just as you would with an ordinary script.

In this exercise, you will learn:

- How to debug Web Workers by passing data
- How to debug Web Workers using Internet Explorer’s integrated debugger

#### Task 1 - Debugging Web Workers by passing data ####

One of the most common debugging techniques entails writing debug information to the browser’s console using the **console** object. However, the **console** object is defined on the **window** object, which is not available to the Web Worker. Working around this limitation is not difficult. The simplest way to write to the console is to pass debug information back to the main script so the main script can write to the console.

For this task, we need to solve a bug in which one of the image filters - the inversion filter - is not working; clicking the **Invert** image shows an alert dialog with the message “No filter was applied”. We know the problem is in the **ImageHandler.Filters.js** file in the **Scripts** folder, which is the Web Worker script that performs the image manipulation computations in the background, but we do not know what is causing the bug.

1. Open the **NewGame.htm** file in Internet Explorer 10.
1.	In Internet Explorer, add a new image or open an existing game. When an image is open, click the **FILTERS** menu item followed by the **Invert** filter (third filter from the left). Close the alert dialog. Notice that nothing happens to the image.
1.	Leave Internet Explorer open and switch to your editor. Open the **ImageHandler.Filters.js** file in the **Scripts** folder. In the event handler for the **message** event there is a **switch** statement that determines which filter to apply to the image. As no filter is applied, we want to check if this event is fired and verify the correct filter is called. Add the following statement to the first line of the event handler (the function that is bound to the message event).

	<!-- mark:2 -->
	````JavaScript
	self.addEventListener('message', function (e) {
		self.postMessage({ log: e.data.effect });
		var data;
	````

	This statement sends a message containing the data we want to log to the main script. We want to print the value of the **e.data.effect** property because this is the property evaluated by the **switch** statement.

	> **Note:** This code creates a JavaScript object and sends the object to the worker’s event listeners. It is important to note that the browser automatically serializes and deserializes the message, creating a clone for each event listener. This has two important ramifications: the objects passed to the **postMessage** method are not shared between the worker script and the main script; and you can send more complex data structures, as long as they can be serialized and deserialized by the browser.

1.	Open the **ImageHandler.js** file in the **Scripts** folder and find the **_applyFilter** method (note the underscore). 

1. Locate the line of code in which the Worker object is created. The following line adds an event handler for the **message** event using the **addEventListener** method. This event handler is called when the worker calls the **postMessage** method in the previous step, so we need to respond appropriately when the message contains log data. Inside the event handler function, add an “if” block that checks if the **e.data.log** property exists and then outputs the property’s value to the browser’s console. The event handler should look like this:

	<!-- mark:7-10 -->
	````JavaScript
	worker.addEventListener('message', function (e) {
		 if (e.data === null || e.data === undefined) {
			  alert('No Filter was applied');
			  return;
		 }
		
		 if (e.data.log) {
			  console.log('The worker says: ' + e.data.log);
			  return;
		 }

		 ctx.putImageData(e.data, 0, 0);
	}, false);
	````

	When the **e.data.log** property exists, the message is written to the **console** object. We could just as easily have used an **alert** dialog to show the message, but using an alert has a big disadvantage. When you debug your applications in this way, you may need to output a significant amount of debug info or multiple log messages. Showing the messages in a dialog becomes unwieldy and inconvenient. It is also much easier to selectively copy information that appears on the console to the clipboard for further inspection.

1.	Save all files.

1.	In Internet Explorer, press **F12** to open the Internet Explorer Developer Tools, if not already open. If the Developer Tools pane opens in a separate window, click the **Pin** icon at the top right to attach it to the main browser window. Click the **Console** tab in the Developer Tools pane.

1.	Press F5 and then repeat steps 1 and 2 to see the results of our changes. Close the alert and check that the posted log message was written to the console. The console now contains the text: “**The worker says: invert**”.

1. Evaluate the log data. The message means that the large switch statement contains a case whose value is “**invert**”. Open the **ImageHandler.Filters.js** file in the editor and look for a line that reads **case 'invert'** in the switch statement. This line cannot be found, but there is a line that reads **case 'inversion'** instead. It appears that we have discovered the root cause of the bug: a simple mismatch. Change the text from “inversion” to “invert”:

	<!-- mark:1 -->
	````JavaScript
	case 'invert':
		data = applyFilter(e.data.imageData, invert);
		break;
	````

1.	Save all the files and repeat steps 1 and 2. Check that the filter is now applied correctly.

2.	Now that the bug is fixed, clean up by removing the code changes made in step 3. Posting extraneous messages can hurt performance and make it difficult to resolve newly discovered bugs.

#### Task 2 - Debugging Web Workers with the Integrated Debugger ####

Being able to log information to the console is very helpful, but not all problems are solvable in this way. For example, if we wanted to debug one of the filters, it might be difficult to figure out what the problem is without stepping through the code that applies the requested effect line by line.
 
Internet Explorer 10’s integrated debugger allows us to debug worker scripts interactively. However, because the worker scripts run in a separate environment, we have to instruct Internet Explorer to attach to the running worker script using the **debugger** statement.

1.	In your editor, open the **ImageHandler.Filters.js** file in the **Scripts** folder.

1. Add the **debugger** statement to the first line of the **message** event handler function.

	<!-- mark:2 -->
	````JavaScript
	self.addEventListener('message', function (e) {
		debugger;
		var data;
		/* ... */
	}, false);
	````
1. Save your changes and return to Internet Explorer 10.

	> **Note:** If you are using Visual Studio to run the application, do not press **F5** to run the application because Visual Studio will attach its own debugger instead of Internet Explorer’s. Either switch to the browser on your own or press **Ctrl+F5** to run the application without attaching the debugger.

1.	In Internet Explorer 10, open the Developer Tools pane, if not already open, by pressing **F12**.

1.	Switch to the **Script** tab on the Developer Tools pane and click the **Start debugging** button.

1.	Add a new image or open an existing game. When an image is open, click the **FILTERS** menu item and click one of the filters.

1. Internet Explorer immediately opens the **ImageHandler.Filters.js** file in the **Script** tab of the Developer Tools, highlights the line with the debugger statement, and places the yellow arrow denoting the current statement in the gutter area.

	![Image 85](images/ie10-integrated-debugger.png?raw=true)

	_Internet Explorer 10's Integrated Debugger_

1.	You can now walk through the code as you would in Visual Studio or another IDE. You can set breakpoints by clicking on the gutter next to the line where you want to place the breakpoint. You can press F10 or F11 to step over or into your code, respectively, or F5 to continue running until the next **debugger** statement or breakpoint. In addition, the debugger is fully interactive, so you can write statements and expressions in the console on the right, add watches, and modify values on the fly.

1.	Place the mouse over the word **effect** in the **switch** statement and observe the debugger tooltip showing the name of the selected filter.

1.	Right-click the word effect and select **Add watch** from the context menu. Internet Explorer adds the expression **e.data.effect** to the **Watch** tab on right.

1.	Press F10 and F11 a few times to walk through the code.

1.	To end the debug session, click the **Stop debugging** button.

1. Only use the **debugger** statement in your development environment to debug code. Never use it in production code, so ensure it is cleaned up. Using your editor, remove the **debugger** statement from the **ImageHandler.Filters.js** file.

<a name="Exercise2" />
### Exercise 2: Using the Compat Inspector ###

Web developers are accustomed to testing their sites on multiple browsers and using browser or feature detection techniques to ensure they work well on all target browsers. Prior to HTML5, the most popular browsers interpreted the HTML standard differently in many cases, or implemented different feature sets, which made these techniques all the more necessary. However, HTML5 helps developers avoid many of the difficulties associated with cross-browser support. Browser vendors, Microsoft among them, are working together to ensure they interpret and implement the standard in the same way and that code written for one browser works and looks the same on other browsers.

Although modern browsers may be more compatible with each other than ever before, many of these improvements break compatibility with older browsers. As a result, sites built for previous versions of Internet Explorer may need to be updated to take advantage of the improved cross-browser support.

To help you determine what site changes you should make due to the improvements in Internet Explorer 10, Microsoft has developed a tool called **Compat Inspector**. It is a JavaScript library that analyzes your site for common compatibility issues and helps you quickly resolve them. When the Compat Inspector runs, it displays errors and warnings, and provides other improvement suggestions. In addition, it can automatically emulate many of the changes it recommends without modifying your code, to help you determine the extent of the changes.

#### Task 1 - Fixing a Compatibility Error Using the Compat Inpsector ####

Some of the changes to Internet Explorer 10’s rendering engine break HTML and JavaScript code written for previous versions of Internet Explorer. These changes were made to comply with the HTML5 standard and to ensure Internet Explorer 10 behaves like other browsers. This task shows how to use the Compat Inspector to find breaking changes and correct them to work with Internet Explorer 10.

1.	Browse to the **NewGame.htm** page in Internet Explorer 10.

1.	Add a new image or open an existing game.

1.	Click the **DONE** button on the bottom menu to view the final image in a popup.

1. Notice that an error occurs and that the final image is not displayed.
 
	![webpage error](images/webpage-error.png?raw=true)

	_The Internet Explorer error dialog showing the INVALID_CHARACTER_ERR error._

	The error shown in the error dialog is INVALID_CHARACTER_ERR.

	> **Note:** In case you don’t see the error popup, please following steps to enable the JavaScript debugger:

	> 1. On the Tools menu, click Internet Options.
	> 2. In the Internet Options dialog box, click the “Advanced” tab.
	> 3. In the Browsing category, clear the “Disable script debugging” check boxes.
	> 4. Click OK.
	
1. Click **Yes** in the error dialog to open Internet Explorer’s script debugger. The script debugger automatically enters debug mode and highlights the line causing the error:

	![IE 10 script debugger error](images/ie-10-script-debugger-error.png?raw=true)

	_The Internet Explorer 10 Script Debugger highlighting the line where the bug occurs_ 

	There appears to be nothing wrong with the line. Syntactically, it is correct. This line of code works well in previous versions of Internet Explorer.
	
1.	To resolve this problem, we can use the Compat Inspector. Return to your editor and open the **NewGame.htm** file.

1. Add a new script element that references the Compat Inspector script. Make sure to add the new script before all the other script elements. Otherwise, it will not work as intended.

	<!-- mark:4 -->
	````HTML
	<head>
		<title>The FacePlace</title>
		<link href="CSS/Base.css" rel="stylesheet" type="text/css" />
		<script src="http://ie.microsoft.com/testdrive/HTML5/CompatInspector/inspector.js"></script>
		<script src="Scripts/jquery-1.7.1.js"></script>
	````
1. Return to Internet Explorer 10 and refresh the page. Note the three numbers in the colored boxes in the top right corner.

	![The Compact Inspector Summary Buttons](images/the-compact-inspector-summary-buttons.png?raw=true)

	_The Compat Inspector summary buttons_ 

	These boxes provide an overview of the number of errors, warnings, and suggestions that Compat Inspector can offer at this time. When first opening the site, the Compat Inspector detects zero errors, four warnings, and two additional suggestions.

1.	Repeat steps 2 and 3 to reproduce the problem. Click **No** to close the error dialog.

1.	Notice that the red box showing the error count now shows one error.

1. Click the boxes to see a list of the Compat Inspector’s messages. Scroll until you find the error (with the red band on the left).	

	![Compat Inspector Message List](images/compat-inspector-message-list.png?raw=true)

	_The Compat Inspector message list_ 

	Read the message. Apparently, creating elements using the angle brackets is no longer support when calling the **createElement** method. Notice that this change has been made to comply with the standard and other browsers. You can click the links to read more about the error and find ways to resolve the problem. Clicking the message also opens the message in the **Details** page and displays more information about it.

	> **Note:** On the Tests page, you can select which tests you want the Compat Inspector to run. You might want to disable tests that you know do not affect your site. For example, for the **TheFacePlace** application, the Compat Inspector shows warnings about possible browser sniffing (usually a bad practice). However, the calls that triggered these warnings are made by the jQuery library and not by the application’s code, so these tests can be safely removed in this session.
	
1.	Before we fix the problem, we want to check if simply removing the angle brackets resolves the problem. Compat Inspector can emulate the corrected behavior without changing the code or leaving the browser. Click the **Verify** checkbox and then press **F5** to refresh the page.

2.	Repeat steps 2 and 3 to check if the problem persists. Notice that this time, clicking the **DONE** button does not produce an error. Instead, the final image displays correctly in the popup, indicating that the Compat Inspector’s suggestion works.

1. In your editor, open the **NewGame.js** file in the **Scripts** folder. Search for the line that contains the string **“`<img>`”** (without the quotes). This line produces the error and is highlighted in Internet Explorer’s script debugger (in step 5). Remove the angle brackets:

	````JavaScript
	var img = document.createElement("img");
	````

	> **Note:** This bug fix is simple and mostly backwards compatible. This is not always the case. For more complex bugs, or fixes that may break support for older target browsers, you may have to use additional techniques, such as feature detection, to determine where to apply the fix. In most cases, the fix for Internet Explorer 10 applies to other HTML5-compliant browsers as well.

1.	In the **NewGame.htm** file, remove the Compat Inspector script by deleting or commenting out the line added in step 7. This helps ensure that we are testing our changes and not Compat Inspector’s emulation. It is also very important to remember to remove the Compat Inspector before moving the code to a production environment.

1.	Save the files, return to Internet Explorer, and refresh the page. Ensure the Compat Inspector summary boxes no longer appear.

1. Repeat steps 2 and 3. Notice that the image displays in a popup, as expected.

## Summary ##

Internet Explorer 10 aims to make the developer’s work easier by improving compatibility and providing better tools. Web workers are fully supported, help improve the perceived application performance (and thereby the user experience), and now, thanks to the improved F12 Developer Tools, can be debugged very easily inside the browser. Microsoft also provides additional tools such as the Compat Inspector to help developers update their applications for modern browsers like Internet Explorer 10.