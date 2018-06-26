---
layout: post
title: OT - A quick post on using gpg to manage github in Linux
comments: true
---

After being used to OSX Keychains in my old mac, I wanted a similar
functionality on my Linux laptop. Being prompted each time to login,
while interacting with [github](https://github.com) for instance was
annoying. Searching the web, led me to
[git-credential-store](https://git-scm.com/docs/git-credential-store).
I was not too happy with the fact that credentials would be stored in
clear text, locally, indefinitely. I came across this stackoverflow
[post](http://stackoverflow.com/questions/5343068/is-there-a-way-to-skip-password-typing-when-using-https-on-github/18362082#18362082) that
I adapted for Linux. The concept is as follows:

   1. Use a .netrc file to store your credentials. This follows the
      file format for [netrc](https://www.ibm.com/support/knowledgecenter/SSB27U_6.2.0/com.ibm.zvm.v620.kijl0/netrcd.htm#netrcd)
   2. Encrypt this file,
      using [GNU Privacy Guard](https://www.gnupg.org/)
<!--more-->
   3. Download and add the
      [git credential helper script](https://raw.githubusercontent.com/git/git/master/contrib/credential/netrc/git-credential-netrc) to
      your path.
   4. Update git config to tell it to use the encrypted credentials file

### STEP 1: The .netrc file
As described in the link above, the .netrc file follows a standard and
looks like this: [![](/assets/gpg3.png)](/assets/gpg3.png)

We save this in our home directory - this is obviously clear-text. We will now
encrypt the file and delete the clear-text one

### STEP 2: Encrypt the .netrc file
GNU Privacy Guard, was new to me. Solving this specific problem also
taught me about how to use GPG in general to sign and encrypt any
piece of data that needs to be exchanged securely. The basic premise
is we create a public/private key pair, similar to SSH. This is stored
in a "key-ring" on our computer. We can potentially add other trusted
public-keys to this key-ring and decrypt/verify files that are signed
by the sender's private key.
For our example, here is how I did it:

_GENERATE THE KEY PAIRS_

[![](/assets/gpg1.png)](/assets/gpg1.png)

_After moving the mouse around for a bit:_

[![](/assets/gpg2.png)](/assets/gpg2.png)

_You can see the keyring :_

[![](/assets/gpg4.png)](/assets/gpg4.png)

_Now we can use our private key to encrypt the .netrc file_

[![](/assets/gpg5.png)](/assets/gpg5.png)

(I had to specify a recipient. In our case since github was going to
use my email id as the 'end-user' to decrypt the .netrc file, I
specified it as the recipient)

At this point, you may delete your clear text .netrc and will be left
with a .netrc.gpg file

### STEP 3: Download and add the git credential helper:
[This perl script](https://raw.githubusercontent.com/git/git/master/contrib/credential/netrc/git-credential-netrc) is
used by git to unencrypt the .netrc.gpg file. So you will need to make
it executable and store it in a directory in your ```$PATH```
variable.

``` shell
curl -o /home/ajay/bin/git-credential-netrc https://raw.github.com/git/git/master/contrib/credential/net‌​rc/git-credential-ne‌​trc
```

### STEP 4: Tell git about the encrypted credentials
Finally, you use the git config command to inform git to use the
credential helper. This is when you also point it to the location of
the newly encrypted .netrc.gpg file.


``` shell
git config --local credential.helper "netrc -f /home/ajay/.netrc.gpg -v -d"
```

( The -v and -d are diagnostic flags - verbose/debug. This helps you
track issues if you run into any. )



If you happen to follow these steps and if you had any questions, I
would be happy to help and hopefully better my understanding as well.
