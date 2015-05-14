# What is WAPA?

WAPA stands for Web Accessibility Properties and Actions.  WAPA is a technology that allows scripted web applications more access to the accessibility layer.
##Why do we need it?
* ARIA is a one-way API.  It only allows web developers to set properties in the accessibility API via markup.  It does not allow the platform accessibility API (UIA on Windows) to send actions to UI created by the web developer. WAPA allows the accessibility API to trigger an action.
* Platform Accessibility API properties, such as the checked state of a checkbox, can come from native HTML, ARIA or be maintained in javascript objects.  Tracking their state is very confusing.  WAPA unifies the state into a single place.
* Some javascript apps don’t use DOM attributes to store state at all, and traditional ARIA does not work for them
* WAPA allows the ARIA attributes to be added in response to an Accessibiltiy API call, rather than for all users.  This reduces download cost.  For applications that change the DOM a lot, it can also improve performance by reducing the number and size of changes.
