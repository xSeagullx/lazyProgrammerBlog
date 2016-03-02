---
layout:     post
title:      "Language injections in IntelliJ IDEA"
date:       2016-03-02 23:20:00
summary:    I'll show how to simplify modification of code fragments, that are kept in files of different type.
            Things like Groovy or CSS code in XML file, Java class reference in tag, or SQL in string are not a
            reason to deny yourself of autocompletion and smart highlighting.

categories: intellij language-injection tips
---

Sometimes you might need to edit code fragment located inside source file of another language.
IntelliJ IDEA have a nice feature, allowing to inject custom language into given block.

Imagine we want to keep CSS in xml (don't ask me why).
Here how it will looks. Not nice at all.

![plain xml](/assets/2016-03-02-intellij-language-injection/img1.png)  
  
Just add a comment `<!-- language=CSS -->` before the tag, and see that now you have highlighting, autocompletion,
inspections and much more!

![comment](/assets/2016-03-02-intellij-language-injection/img2.png)  
  
If you need to make this changes global, just call a context menu, and choose "inject language or reference menu".

![shortcut](/assets/2016-03-02-intellij-language-injection/img4.png)  
  
You can achieve same results with a bit more control via settings.

![menu](/assets/2016-03-02-intellij-language-injection/img3.png)  

As you can see from previous screen, IDEA already have dozens of injections configured for you, use them as an inspiration.
Personally I use it when I'm editing __BPMN__ files with Groovy code inside, and xml with XPath expressions in tags.

Enjoy and experiment!

P.S. If it doesn't work, check that __IntelliLang__ plugin is enabled!
More details: [in documentation](https://www.jetbrains.com/idea/help/using-language-injections.html)
