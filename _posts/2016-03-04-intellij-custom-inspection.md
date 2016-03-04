---
layout:     post
title:      "Custom inspections in IntelliJ IDEA"
date:       2016-03-04 22:20:00
summary:    "Configuring custom inspections in IntelliJ IDEA with Structural Search and Replacement inspection"

categories: intellij tips
---

Consider following example, which might be a bit artificial, as it's just an illustration.

![exampleCode](/assets/2016-03-04-intellij-custom-inspection/exampleCode.png)

We want IDEA to detect and allow us to transform first line to second one.

Just like this:

![inspectionAction](/assets/2016-03-04-intellij-custom-inspection/inspectionAction.png)

Awesome, yes? 

<p class="bs-callout bs-callout-primary">Note, that it's not a dummy regexp replace. Data is __context-specific__, i.e. you will not trigger Java inspection for text being in comments or string literal.  
It doesn't matter if you import class or use fully qualified name, IDEA will be able to trigger inspection.</p>

### So, here how to setup this.


First - open the settings and enable  

![setup1](/assets/2016-03-04-intellij-custom-inspection/setup1.png)

Then you'll have to describe what are you looking for.
<p class="bs-callout bs-callout-warning">In case you have fixed class name in your template, e.g. static method or cast, you may want to specify it with fully qualified name. It will limit inspection scope to particular class, and prevent possible clash.</p>  


![setup2](/assets/2016-03-04-intellij-custom-inspection/setup2.png)

You see __Edit variables__ button. It opens a dialog window, that gives you a several ways to restrict variable contents. It's done via regular expressions.  

<p class="bs-callout bs-callout-info">If you have several inspections capturing same piece of code, replacements for all of them will be proposed in context menu.  
Proposal order will be the same as it is displayed in settings.
Move more common action on top, and you'll save yourself few keypresses.</p>

Give it a try, and make your code better or enforce some custom conventions in your code!

For more information consult [official documentation](https://www.jetbrains.com/idea/help/structural-search-and-replace-examples.html).
