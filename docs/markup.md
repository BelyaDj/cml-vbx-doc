# Grid Tables

Instruction is actual for Ubuntu 14.04. Python 2.7
 
To install third party extension for MarkDown for grid tables (as in ReStructuredText):

0) Find location on markdown package (by default in /usr/local/lib/python2.7/dist-packages/markdown)
```sh
# find \ -name 'markdown'
```
    
1) Download extension for grid tables
```sh
# git clone https://github.com/smartboyathome/Markdown-GridTables.git
```

2) Copy extension to MarkDown package
```sh
# cp Markdown-GridTables/mdx_grid_tables.py /usr/local/lib/python2.7/dist-packages/markdown/extensions
```

3) Enable extension: in mkdocs.yml add string to 'markdown_extensions'
```
markdown_extensions:
  - mdx_grid_tables
```
  
If you made all right you'll see table with lists in sells below:

+---------------+---------------+--------------------+
| Fruit         | Price         | Advantages         |
+===============+===============+====================+
| Bananas       | $1.34         | - built-in wrapper |
|               |               | - bright color     |
+---------------+---------------+--------------------+
| Oranges       | $2.10         | - cures scurvy     |
|               |               | - tasty            |
+---------------+---------------+--------------------+

__About built-in extensions:__ <https://pythonhosted.org/Markdown/extensions/index.html>        
__List of third-party extensiond:__ <https://github.com/waylan/Python-Markdown/wiki/Third-Party-Extensions>


# Markdown Markup

[TOC]

---

## Admonition Examples

---

```
!!! note
    Ba-da Bums...
```

!!! note
    Ba-da Bums...

---

```
!!! note "Ba-da Bums..."
    Ba-da Bums...
```

!!! note "Ba-da Bums..."
    Ba-da Bums...

---

```   
!!! important
    Ba-da Bums...
```

!!! important
    Ba-da Bums...

---

```
!!! danger
    Ba-da Bums...
```

!!! danger
    Ba-da Bums...

---

```
!!! type "optional explicit title within double quotes"
    Ba-da Bums...
```

!!! type "optional explicit title within double quotes"
    Ba-da Bums...

---

```
!!! important ""
    Ba-da Bums...
```

!!! important ""
    Ba-da Bums...

---
