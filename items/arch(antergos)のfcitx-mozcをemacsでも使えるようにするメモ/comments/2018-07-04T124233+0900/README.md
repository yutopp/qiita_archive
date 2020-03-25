(自前のArch Linux と Manjaro Linuxで試したところ)、不明な公開鍵についてのエラーが出るようです。
なのでその際には、
    
    $ yaourt -P fcitx-mozc -i fcitx-mozc
       ...
    ==> gpg でソースファイルの署名を検証...
    fcitx-mozc-2.18.2612.102.1.patch ... 失敗 (不明な公開鍵 <keyid>)

    $ sudo pacman-key --recv-keys <keyid>
    $ gpg --recv-keys <keyid>
    $ yaourt -P fcitx-mozc -i fcitx-mozc

とすると良いです。
調べればすぐ出てくるかも知れませんが、念の為メモとしてコメントを残させて下さい。
