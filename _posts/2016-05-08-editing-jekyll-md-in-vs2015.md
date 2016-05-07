---
layout: post
title: Editing Jekyll markdown pages in Visual Studio 2015
date: 2016-05-08 18:00:00 +0200
tags: [visual studio]
---
*nix-like OSs are gaining much popularity even in .NET world since Microsoft decided to move on with their products into this direction (you can read more on [Hanselman's](http://www.hanselman.com/blog/DevelopersCanRunBashShellAndUsermodeUbuntuLinuxBinariesOnWindows10.aspx) [blog](http://www.hanselman.com/blog/AnUpdateOnASPNETCore10RC2.aspx)). Nevertheless it's still more convenient to develop .NET apps on Windows (especially if it's your target platform).

If you want to play with [Jekyll](http://jekyllrb.com) on Windows I recommend you a great [step-by-step manual](http://jekyll-windows.juthilo.com) written by [Julian Thilo](https://twitter.com/juthilo). It covers many pitfalls that are patiently waiting for any Windows user that never had anything to do with Ruby framework.

Unfortunately there is still one more pitfall for Visual Studio (2015) users who, like me, would like to edit their posts directly on their IDE of choice. Jekyll expects all the input files (files to be transformed by Liquid) in UTF-8 **without BOM** ([byte order mark](https://en.wikipedia.org/wiki/Byte_order_mark)). If you have any HTML or Markdown input file in your site folder it will be just copied as is to the target directory (`_site` by default) without any error or warning. 

And here Visual Studio has one very annoying behavior which is adding BOM by default to all UTF-8 encoded files on save. You can change this for specific file by using `File > Advanced save options...` but doing it for every single new file is even more irritating than manual conversion in other editor (like Notepad++). If it's your use case I suggest installing one very useful VS plug-in - [Fix File Encoding](https://visualstudiogallery.msdn.microsoft.com/540ac2d8-f881-4794-8b00-810d28257b70) by Sergey Vlasov. Just don't forget setting-up the proper file extensions in `Tools > Options... > Fix File Encoding > General` for both HTML and Markdown files:

`
\.(htm|html|md|markdown)$
`

Another handy plug-in is [Markdown Mode](http://visualstudiogallery.msdn.microsoft.com/0855e23e-4c4c-4c82-8b39-24ab5c5a7f79) which provides syntax highlighting and HTML preview for Markdown files. There is also a good spell checking add-on for VS - [Visual Studio Spell Checker](https://github.com/EWSoftware/VSSpellChecker/wiki) - which really does the job.

Cheers and happy writing! 
Marek