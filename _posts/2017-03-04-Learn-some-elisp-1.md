---
layout: post
title: "Learn some elisp [1]"
date: 2017-03-04
---

`directory-file-name`: Remove the file slash in Unix-syntax

`file-name-directory`: Get the directory of file

In emacs, directory is also handled like file.
The way to get parent directory:


```elisp
(defun parent-directory (dir)
  (unless (equal "/" dir)
    (file-name-directory (directory-file-name dir)))
  )
```

`with-temp-buffer`: Create a temporary buffer

`insert-file-contents`: Insert contents of file after point

`split-string`: split string into substrings

`thing-at-point`: return the ting at point

Below function get value of specified attribute name from file.
The structure in the file like
attributeName: attributeValue


```elisp
(defun get-attribute-from-mi-file (mi-file attrName)
  (with-temp-buffer
    (insert-file-contents mi-file)
    (let ((moreLines t)
          (attrFound nil)
          (lineToken '()))
      (while (and moreLines (not attrFound))
        (setq lineToken
              (split-string (thing-at-point 'line t) ":" t split-string-default-separators))
        (when (and (>= (length lineToken) 2) (equal (car lineToken) attrName))
          (setq attrFound t)
          )
        (setq moreLines (= 0 (forward-line 1)))
        )
      (when attrFound
        (car (last lineToken))))
    )
  )
```

`save-excursion`: Save point, and current buffer

`search-backward`: Search backward from point for string

`backward-char`: Move point backword

Below function try to fetch the class name of toString method in C++ style.
This function only expect that the implementation of toString is out of class
declaration.

```elisp
(defun fetch-tostring-classname (pos)
  (save-excursion
    (when (search-backward "::toString()" nil t)
      (backward-char 2)
      (thing-at-point 'symbol))
    )
  )
```

`file-name-as-directory`: Just appends a slash for a Uix-syntas file name

`concat`: concatenate strings


Below function concat one directory with several directory or file names.
(concat-path '/opt/apps/' 'a' 'b' 'c') returns /opt/apps/a/b/c


```elisp
(defun concat-path (directoryName fileName &rest otherFileNames)
  (let ((concatPath (concat directoryName fileName)))
    (while otherFileNames
      (setq concatPath (concat (file-name-as-directory concatPath) (car otherFileNames)))
      (setq otherFileNames (cdr otherFileNames)))
    concatPath
    )
  )
```

`string-match`: Get the index of start of first match, or nil

`match-string`: Get the string of text matched by last search.

Below function extract one string from another string, and assume that the original string has the defined format. the input string like foo/barCT, the return string is bar.


```elisp
(defun get-ct-component (ctIdentity)
  (string-match "\\w+/\\(\\w+\\)CT" ctIdentity)
  (match-string 1 ctIdentity)
  )
```

`with-current-buffer`: Keep current buffer, execute on other buffer

`start-process`: start a programm in a subprocess.

`pop-to-buffer`: Select buffer in some window, perferably a different one.

`erase-buffer`: Delete the entire contents of the current buffer.

`get-buffer-create`: Get the buffer specified by buffer name, createing a new one if needed.

Below function create one buffer `ct-run` if not exist, clear the buffer. Execute program ct\_run output to that buffer, then switch to that buffer.


```elisp
(defun mi-ct-run ()
  (interactive)
  (with-current-buffer (get-buffer-create "*ct-run*") (erase-buffer))
  (start-process "ct-run" "*ct-run*" "ct_run")
  (pop-to-buffer "*ct-run*"))
```
