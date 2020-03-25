ニャン :cat2:

## 1. mozc-emacs, mozcがインストールされている場合はアンインストールしておく

`fcitx-mozc`とconflictするため．

```bash
sudo pacman -R emacs-mozc mozc
```

## 2. fcitx-mozcのPKGCONFIGを落としてくる

```bash
yaourt -G fcitx-mozc
```

## 3.パッチを当てる

```diff:PKGBUILD.patch
--- PKGBUILD	2018-01-08 16:08:09.000000000 +0900
+++ -	2018-01-13 21:17:48.133846572 +0900
@@ -77,7 +77,7 @@
 
   cd mozc/src
 
-  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx/fcitx.gyp:fcitx-mozc"
+  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx/fcitx.gyp:fcitx-mozc unix/emacs/emacs.gyp:mozc_emacs_helper"
 
   QTDIR=/usr GYP_DEFINES="document_dir=/usr/share/licenses/$pkgname use_libzinnia=1" python2 build_mozc.py gyp
   python2 build_mozc.py build -c $_bldtype $_targets
@@ -91,6 +91,8 @@
   install -D -m 755 out_linux/${_bldtype}/mozc_server "${pkgdir}/usr/lib/mozc/mozc_server"
   install    -m 755 out_linux/${_bldtype}/mozc_tool   "${pkgdir}/usr/lib/mozc/mozc_tool"
 
+  install -D -m 755 out_linux/${_bldtype}/mozc_emacs_helper "${pkgdir}/usr/bin/mozc_emacs_helper"
+
   install -d "${pkgdir}/usr/share/licenses/$pkgname/"
   install -m 644 LICENSE data/installer/*.html "${pkgdir}/usr/share/licenses/${pkgname}/"
 
```

```bash
patch fcitx-mozc/PKGBUILD PKGBUILD.patch
```

## 4. ビルド&インストールする

```bash
 yaourt -P fcitx-mozc -i fcitx-mozc
```

## 5. mozc.elを入れる

```emacs
M-x package-install mozc
```
