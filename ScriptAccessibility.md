EARLY DRAFT
# Script-Based Web Accessibility

### Abstract
This is a proposal for a set of User Intention Events that build on ARIA to extend accessibility functionality to complex, scripted web applications. It supports use cases where markup-based solutions have proven impossible, difficult or costly to implement.


# Introduction

This proposal is being made to start discussion in the Web Incubation Community Group to start discussion, with the expectation there will be changes as it is combined with similar proposals.

There is an experimental implementation of this in Microsoft Edge. To enable it, launch Edge in Windows 10, go to about:flags, scroll to the bottom, and check "Enable experimental accessibility features."

## Background and Rationale

This proposal is based on [WAPA](https://github.com/cyns/wapa) and got further inspiration from [Indie UI Events](https://w3c.github.io/indie-ui/indie-ui-events.html) and [Mozilla Web Accessibility API](https://asurkov.github.io/a11yapi/)

This proposal addresses the challenge of allowing web applications -- those built with scripts running in browsers -- to use the accessibility features of modern operating systems. Today desktop assistive tools are written for a specific platform an can call the platform-level APIs directly. Accessibility in browsers is enabled mainly by putting markup in web content, but applying this approach to scripted web applications is too difficult to be used effectively in practice.

This Script Based Web Accessibility API allows more direct communication between web applications and platform accessibility APIs. It includes three pieces

1.  A way for web authors to provide ARIA name-value pairs (properties) to platform accessibility APIs from script, without requiring an element with those aria properties in the Document object.
2.  A way for web authors to react in script to method calls from platform accessibility APIs (actions)
3.  A mechanism for handling accessibility information for virtualized content, where the DOM contains only a subset of the actual dataset or document.

## Relationship to Other work

### Relationship to ARIA

Script Based Web Accessibility is intended to extend the functionality of ARIA to situations where it will be easier or more performant to manage accessibility in script rather than in markup. It allows web authors to provide ARIA name-value pairs to the Accessibility API via javascript functions without the need for an element with those name-value pairs to be present in the document object.

### Relationship to IndieUI

[IndieUI Events List](https://dvcs.w3.org/hg/IndieUI/raw-file/default/src/indie-ui-events.html#eventslist) has some overlap with this functionality. Our goal is to harmonize with IndieUI and User Intention Events

### Relationship to User Intention Events

Script Based Web Accessibility is a set of [User Intention Events](https://w3c.github.io/editing/device-independent-events-explainer.html) . It builds on that work to hook into the platform accessibility layer, and has a dependency on that work.

### Relationship to Mozilla Accessibility API

The [Mozilla Web Accessibility API](https://asurkov.github.io/a11yapi/) is an independent effort that addresses some of the same challenges as this Script Based Web Accessibility proposal does. We hope it will also be submitted to the WICG so ideas from these and other proposals can cross-fertiilize and produce a vigorous hybrid that will be implemented and standardized.

## Scenarios

This spec aims to enable the following scenarios:

1.  Performance improvements from Just-in-Time loading of accessibility information

Pre-loading accessibility information make it available to scripts is costly, and of little value to those not using accessibility technologies. For example, populating all the strings with `aria-label` takes time to load those strings into the objects or download them as part of static markup. Similarly, updating ARIA state is costly. For example, if an item is removed from a list of 100, every other item must update its `aria-setsize` and potentially its `aria-posinset` attributes

3.  Accessibility information Stored outside DOM

A web application uses Javascript objects to store all its state (including accessibility state), and only uses the DOM as a drawing surface. Ideally the script could manage accessibility state in the same was as it manages other state, directly from the script.

5.  Virtualized Content

In a wordprocessing app (e.g. Office 365 or Google Docs) the script code works with only a page or two of the document being edited at a time rather than loading the entire document into the browser context. This scenario envisions more content loaded when the user reaches the end of the previously-loaded portion of the document.

7.  Custom Elements (Future)

A developer encapsulates functionality in a control associated with a custom element. Using DARIA, the developer can define default ARIA roles, states, and properties that are mapped to the platform accessibility APIs in the way these are mapped to the standard HTML elements. Script Based Web Accessibility APIs allow more direct communication with the platform accessibility APIs from scripts, including the ability to intercept accessibility API actions. This allows developers to hook up the custom elements to web accessibility APIs so assistive technology products can interact with them.

9.  WebDriver testing of accessibility (Future)

Since Script Based Web Accessibility is an in-browser Javascript API rather than a platform accessibility api, it should be possible to simulate the effect of an assistive technology acting on content by sending the same command events that the browser would send. This would allow WebDriver to exercise accessibility in web apps.


# Proposed APIs


## Web Accessibility Properties

AT Request for Accessible node property via accessibility API

![Swimlane flowchart of ariarequest object](https://rawgit.com/cyns/wapa/master/ariarequestevent.svg)

1.  AT calls Accessibility API via OS requesting the value of a property on the Accessible node
2.  Accessibility API queries browser
3.  Browser gets the DOM element assosicated with the Accessible node
4.  Browser checks whether the attribute exists on the element
5.  If no
    1.  Browser sends DOM event ariarequest
    2.  Javascript catches ariarequest event
    3.  Javascript sets attributeValue for the attributeName
    4.  If cancel default, end
6.  If yes (or fall through)
    1.  Browser gets aria name-value pair from dom
7.  Browser updates accessibility API with name-value pair
8.  Accessibility API returns name-value pair to calling AT

### IDL


                [Constructor(DOMString type, optional AriaRequestEventInit eventInitDict)]
                interface AriaRequestEvent : Event {
                    readonly attribute DOMString attributeName;
                    attribute DOMString attributeValue;
                };
                dictionary AriaRequestEventInit : EventInit {
                    DOMString attributeName = null;
                    DOMString attributeValue = null;
                };



## Web Accessibility Actions

AT Action taken by user

![Swimlane flowchart of command event initiated by AT](https://rawgit.com/cyns/wapa/master/commandevent.svg)"

1.  User takes action on an element using an AT
2.  AT calls accessibility API method on that element
3.  Accessibility API runs that method against browser
4.  Browser sends User Intention Event on that element with the commandType for the action from the accessibility API (See Mapping Table below for example of UIA to CommandType mappings)
5.  Javascript catches User Intention Event
6.  Javascript modifies DOM attributes and/or script representation of object
7.  If canceldefault, end
8.  browser does action on element using default mechanisms

### IDL


                [Constructor(DOMString type, optional CommandEventInit eventInitDict)]
                interface CommandEvent : Event {
                    readonly attribute DOMString commandType;
                    readonly attribute DOMString detail;
                };
                dictionary CommandEventInit : EventInit {
                    DOMString commandName = null;
                    DOMString detail = null;
                };



## Mapping to Accessibility APIS

### Example: CommandTypes to UIA methods

NOTE: Other APIs need to be fleshed out. Most of these would be default action in MSAA.

<table>

<tbody>

<tr>

<th>commandType</th>

<th>UIA method</th>

</tr>

<tr>

<td>Toggle</td>

<td>IToggleProvider::Toggle</td>

</tr>

<tr>

<td>Expand</td>

<td>IExpandCollapseProvider::Expand</td>

</tr>

<tr>

<td>collapse</td>

<td>IExpandCollapseProvider::Collapse</td>

</tr>

<tr>

<td>scrollIntoView</td>

<td>IScrollItemProvider::ScrollIntoView</td>

</tr>

<tr>

<td>Select</td>

<td>ISelectionItemProvider::Select</td>

</tr>

<tr>

<td>addToSelection</td>

<td>ISelectionItemProvider::AddToSelection</td>

</tr>

<tr>

<td>removeFromSelection</td>

<td>ISelectionItemProvider::RemoveFromSelection</td>

</tr>

<tr>

<td>setValue</td>

<td>IValueProvider::SetValue</td>

</tr>

<tr>

<td>scrollHorizontal</td>

<td>IScrollProvider::Scroll</td>

</tr>

<tr>

<td>scrollVertical</td>

<td>IScrollProvider::Scroll</td>

</tr>

<tr>

<td>setHorizontalScrollPercent</td>

<td>IScrollProvider::SetScrollPercent</td>

</tr>

<tr>

<td>setVerticalScrollPercent</td>

<td>IScrollProvider::SetScrollPercent</td>

</tr>

</tbody>

</table>



## User Intention Events

A User Intention event will be fired whenever an action is taking place, such as toggling a checkbox, to allow authors to hook the action and perform any specific actions they need to, such as updating state and UI, with the option to prevent the default behavior (a simulated click, if applicable). For this, a new event will be added:



            [Constructor(DOMString type, optional CommandEventInit eventInitDict)]
            interface CommandEvent : Event {
                readonly attribute DOMString commandName;
                readonly attribute DOMString detail;
            };
            dictionary CommandEventInit : EventInit {
                DOMString commandName = null;
                DOMString detail = null;
            };



The 'detail' payload part of this event is an extra piece of information defining the event, if necessary, such as the value for a SetValue command.

A User Intention event will fire whenever an AT invokes an action via the accessibility APIs:

![Swimlane diagram of command events.  First the Accessibiltiy Tool sends a command to the browser to toggle the checkbox.  Next, the browser fires the onCommand event.  Next, javascript catches that event, runs application script, and returns to the browser. The browser then runs the default behavior if it has not been cancelled. Finally, the browser returns to the accessibility tool.](https://rawgit.com/cyns/wapa/master/commandevent.gif)

# Examples


This section illustrates how the proposed Script Based Web Accessibility API can address some of the use cases listed in the Introduction.



### Performance improvement with Just-in-Time loading of accessibility information

This illustrates how to provide accessibility information only when it is requested by AT via the Accessibility API, and not at the time the DOM is filled in.

This mechanism works with the @role attribute and any @aria- attribute.

#### JavaScript Sample



            <div id="listbox1" role="listbox" onariarequest="HandleRequest(event);">
                <div role="listitem">Item 1</div>
                <div role="listitem">Item 2</div>
                <div role="listitem">Item 3</div>
                    …
                <div role="listitem">Item 100</div>
            </div>
            <script>
                function manageList(event) {
                    var listbox = document.getElementById('listbox1');
                    if (event.target.parentNode == listbox) {
                        if (event.attributeName === "posinset") {
                            event.attributeValue = GetChildId(event.target);
                        }
                        else if (event.attributeName === "setsize") {
                            event.attributeValue = "100";
                        }
                    }
                }
          </script>







### Accessibility information Stored outside DOM

An application uses javascript objects to store all of its state (accessibility and other state), and only uses the DOM as a drawing surface. The goal is to treat accessibility state the same way as other state, and to manage it directly from script.

element.**onariarequest** is a new event that gets fired when an AT requests makes an Accessibility API request that maps to aria information

element.**oncommand** is a User Intention Event

<div class="issue">TODO: update to match API naming changes in User Intention Events</div>

event.**attributeName** is the attribute that changed as part of the event. This is a new attribute we are proposing to add to the DOM Event Object

event.**preventDefault**(); stops the default processing of the event.

element.**NotifyAriaStateChange**(string) toggles a boolean aria state. It is equivalent to element.setAttribute("aria-"+string, true) or element.setAttribute("aria-"+string, false)







#### JavaScript Sample


                var toggleState = false;
                function HandleRequest(event) {
                    var myCheckbox = document.getElementById("myDiv");
                    if (event.attributeName === "checked" && event.target === myCheckbox) {
                        event.attributeValue = (toggleState ? "true" : "false";
                    }
                }
                function HandleCommand(event) {
                    var myCheckbox = document.getElementById("myDiv");
                    if (event.target === myCheckbox && event.commandName === "toggle") {
                        toggleState = !toggleState;
                        UpdateUI();
                        event.preventDefault();
                    }
                    function ToggleMe(){
                        toggleState = !toggleState;
                        var myCheckbox = document.getElementById("myDiv");
                        myCheckbox.NotifyAriaStateChange("checked");
                    }
                }
            </script>
            <div id="myDiv" role="checkbox" oncommand="HandleCommand(event)" onariarequest="HandleRequest(event);" onclick="ToggleMe();">





### Virtualized Content

A user opens a large document in the Word web app. Word adds the first page or two to the DOM, but not the whole document. When the user reaches the end of the loaded portion of the document, more content is loaded.

How will a screen-reader user know:

*   Position in the document
*   Position within a list or table that spans document chunks
*   When there is more content to load and when there is not
*   When content has been loaded
*   When there has been a network or other error preventing content loading

How will an AT product signal to the app that it needs to load more content?

How will an app let AT products know that more content is loading and that focus should not move out of the content container?

Not all content is in the DOM, or the DOM contains little actual meaning for the content (the DOM is just a view). Requiring that the information be available on the DOM objects is hard to impossible depending on the app. This is used by apps like Office Online, where the content is stored in the backend and only a specific view (such as the current page of a document) is actually available in the DOM.

#### JavaScript Sample

                function HandleCommand(event) {
                    var myContainer = document.getElementById(" content");
                    if (event.target == myContainer && event.commandname == "contentLoaded" ) {
                    appendcontent(mycontainer);
                    event.preventdefault();
                }
                function handlecommand(event) {
                    var mycontainer=document.getElementById("content");
                    if (event.target == myContainer && event.commandname == "contentRequested" ) {
                    mycontainer.setattribute("aria-busy","true");
                    event.preventdefault();
                }
                function handlecommand(event) {
                    var mycontainer=document.getElementById("content");
                    if (event.target == myContainer && event.commandname == "contentTimeout" ) {
                    displayerror();
                    event.preventdefault(); />
                }
                function HandleCommand(event) {
                    var myContainer = document.getElementById("content");
                    if (event.target == myContainer && event.commandName == "contentEnd") {
                        //do Nothing
                        event.preventDefault();
                    }

                    <!-- some UI for the page, which is not virtualized -->
                </div>
                <!--
                 This area is the content of the page, only part of which is loaded.
                 More will be loaded when the user reaches the bottom of the page.
                 aria-async is a proposed new aria attribute that marks the area that
                 contains virtualized content
                -->
    <div id=content aria-async="true" oncommand="HandleCommand(event);">
                    <!-- the first 'chunk' of content.  A page, a set of images, etc. -->
                </div>
    <div id=sidebar>
                    <!-- content which is not virtualized -->
                </div>
            </body>

</div>

**aria-async** is new aria-attribute which means that the content is may be virtualized.

<div class="issue">TODO: Think about naming, and relationship with aria-busy and other live region features</div>

Four new User Intentation Events:

*   **ContentRequested** is fired when the application makes a server call to get additional content. This will often be in response to the user reaching the end of the currently loaded content, but could be in response to custom UI.
*   **ContentLoaded** is fired when the content has been returned from the server and added to the DOM.
*   **ContentTimeOut** is fired after n time if no content is returned.
*   **ContentEnd** is fired when the async call returns with no more content



How does the server indicate this? Is that something we need to define, or up to the app?

TODO: Add where the requestcontent comes from


## Open Questions, Issues, and TODO items

1.  Should Virtualization be a separate spec?
2.  How do you know if the platform AAPI call was successful? Windows UIA methods have return values. Do the other AAPIs?</div>

3.  element.NotifyAriaStateChange(string) toggles a boolean aria state. It is equivalent to element.setAttribute("aria-"+string, true) or element.setAttribute("aria-"+string, false)

3. TODO: what about non-bolean aria attributes? Is this actually necessary?

5.  aria-async is new aria-attribute which means that the content is may be virtualized.

6. TODO: Think about naming, and relationship with aria-busy and other live region features

7.  For the 4 new User Intention event ...

*   How does the server indicate this? Is that something we need to define, or up to the app?

9.  TODO: Add where the requestcontent comes from
10.  NOTE: For the Mapping section, CommandType naming needs to be updated to sync better with User Intentions, ARIA and IndieUI. It is currently based on the closely related UIA methods.
