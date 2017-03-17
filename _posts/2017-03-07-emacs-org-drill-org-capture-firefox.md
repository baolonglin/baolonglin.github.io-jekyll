---
layout: post
title: "Emacs org-drill org-capture org-protocol with firefox"
date: 2017-03-07
---

Emacs can be used as flash card tool, I fallowed the page of [org-drill](http://orgmode.org/worg/org-contrib/org-drill.html).
In emacs 25.1, need to install `org-plus-contrib` first. While installing `org-plus-conrib`, may get error `Invalid function: org-babel-header-args-safe-fn`, I use cpaulik's solution: [remove the ob_R.elc](https://github.com/syl20bnr/spacemacs/issues/4618). And I find one emacs plugin [paradox](https://github.com/Malabarba/paradox), which can be used to manage the installed plugins.
In `Incremental reading`, it shows how to integrate `org-drill` with external browser. It's good to integrate `firefox` which is my default browser. I'm running `Arch` linux with windows manager `fluxbox`, it take me some time to make it working. At begining, i follow the [org-protocol](http://orgmode.org/worg/org-contrib/org-protocol.html), but the `gconftool-2` did not work, even i have `gconftool` installed, and have those configuration in my ~/.config, `firefox` always get error `(org-protocol) isn’t associated with any program`. I just wondered how can i configure the firefox, use application I specified. Then i tried use `network.protocol-handler.app.org-protocol`, got the same error. Then I tried to figure out which application controls the default application in my environment. Then I found [Arch default applications](https://wiki.archlinux.org/index.php/Default_applications), but I don't have normal desktop environment, I have `xdg` installed. How to let my installed application use `xdg-open` to open default application for specified mime type? I still don't know when I write is post. At least I tried to add desktop file into `xdg`, and configure the default application for that desktop file, luckly it works. Below is the configuration in my environment:


* emacs configuration


```elisp
(require-package 'org-plus-contrib)
(after-load 'org
  (require 'org-drill)
  (require 'org-protocol)
  )

  (setq org-capture-templates
      `(("w"
         "Capture web snippet"
         entry
         (file+headline "~/test-drill.org" "Notes")
         ,(concat "* Fact: '%:description'        :"
                  (format "%s" org-drill-question-tag)
                  ":\n:PROPERTIES:\n:DATE_ADDED: %u\n:SOURCE_URL: %c\n:END:\n\n%i\n%?\n")
         :empty-lines 1
         :immediate-finish t)
        ))

```

* firefox configuration

add `network.protocol-handler.expose.org-protocol:false` into about:config, details can be found [Register protocol](http://kb.mozillazine.org/Register_protocol)

add one bookmark with url:


```javascript
javascript:location.href='org-protocol://capture://w/'+encodeURIComponent(location.href)+'/'+encodeURIComponent(document.title)+'/'+encodeURIComponent(window.getSelection())
```

* xdg configuration

add file `emacsclient.desktop` in folder `~/.local/share/applications` with content:


```
[Desktop Entry]
Name=Emacs Client
Exec=emacsclient %u
Icon=emacs-icon
Type=Application
Terminal=false
MimeType=x-scheme-handler/org-protocol;
```


add `x-scheme-handler/org-protocol=emacsclient.desktop` in file `~/.local/share/applications/mimeapps.list`, under the section `[Default Applications]`
