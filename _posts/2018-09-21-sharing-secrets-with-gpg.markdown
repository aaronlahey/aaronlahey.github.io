---
layout: post
title: "Sharing Secrets with GPG"
authors: ["aaron-lahey"]
date: 2018-09-21
tags: ["Coding", "Tools"]
---

As software developers we're often given access to various pieces of sensitive information in order to do our job. This could be anything from database passwords, API keys, TLS certificates, or even private SSH keys to our servers. Also, there's likely going to come a time when we need to share these secrets with our colleagues who don't have them. Maybe they're new to the team, or maybe they're working on a story which requires this information for the first time. Either way, this puts us in a pickle!

It's tempting to simply attach this information to an email or paste/upload it to Slack and assume that everything will be ok. But will it? How can we be sure? Is it worth risking our job, our company's reputation, or our users' privacy?

This blog is meant to be a simple introduction to `gpg` and a walkthrough of how to get set up, generate your own encryption keys, and send secrets over insecure forms of communication. It assumes a working knowledge of the command line and enough of an understanding of your operating system's package manager to install programs.

## Public-key Cryptography
GPG uses a form of cryptography called [public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography). Cryptography is a complex topic, but fortunately for us, a deep understanding of the math and theory behind it isn't necessary to grasp how it works at a high level.

In public key cryptography we generate two encryption keys. One of these keys is called our _private_ key which, as the name suggests, we want to keep a secret. The other is our _public_ key which we can publish for the world to see. These two separate keys let us do some interesting things. First, we can encrypt data using our private key, send it to someone, and using our public key they can decrypt the message and gain a reasonable level of assurance that it was actually us who authored the message. We call this process _signing_. Similar to signing a document using our written signature, we're signing the message using our cryptographic signature. Conversly, anyone in the world can encrypt some data with our public key, send it back to us, and have a reasonable level of assurance that _only we_ can decoded it. We refer to this is as _encryption_ (even though they're both technically encryption).

## Installing GPG
GPG can be installed using [GPG Tools](https://gpgtools.org/) or from the package manager for your specific operating system. Here are some examples...

### macOS
{% highlight bash %}
brew install gnupg
{% endhighlight %}

### Ubuntu
{% highlight bash %}
sudo apt-get install gnupg
{% endhighlight %}

## Generating Your Own Keypair
Although one doesn't technically need their own public and private keys to encrypt a message to someone else, it's a good idea to generate them. We'll need them if we want to receive encrypted messages at some point in the future, and we'll need our own private key for one small step later in the process. To generate a key run the following command...

{% highlight bash %}
gpg --full-generate-key
{% endhighlight %}

This command will start the key generation process and prompt you with a few questions...

{% highlight bash %}
gpg (GnuPG) 2.2.8; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
{% endhighlight %}

The default selecion of `RSA and RSA` is ok here.

{% highlight bash %} RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
{% endhighlight %}

Type `4096` and press enter.

{% highlight bash %}
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
{% endhighlight %}

Let's pick a reasonable expiration time of 1 year by typing `1y` and pressing enter.

{% highlight bash %}
Key expires at Sat Oct  5 20:35:44 2019 CDT
Is this correct? (y/N)
{% endhighlight %}

Type `y`, press enter, and then proceed to fill out the requested information...

{% highlight bash %}
GnuPG needs to construct a user ID to identify your key.

Real name: Aaron Lahey
Email address: alahey@8thlight.com
Comment:
You selected this USER-ID:
    "Aaron Lahey <alahey@8thlight.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
{% endhighlight %}

Type `O` and press enter. When presented with the password screen, type a password and press enter. This is the password you'll use to unlock your private key when you want to sign or decrypt anything.


## Creating Our Secret
Before we begin encrypting files we first need something to encrypt. Let's create a text file with some fake sensitive data using the following command...
    
{% highlight bash %}
$ echo 'secretpassword' > password.txt
{% endhighlight %}
    
Once the file is created lets make sure it's correct...

{% highlight bash %}
$ cat password.txt
secretpassword
{% endhighlight %}

Looks good! Let's imagine I want to send this file to my new teammate Ben. I might be tempted to attach it to an email or upload it to Slack, but remember, this file contains sensitive information--perhaps a production database password! We need a way to encode this file so that Ben is the only one who can decode and read it.

## Importing a Public Key
To do that, we encrypt the file using Ben's public key. Since Ben possesses the corresponding private key and he's taken precautions to keep it safe, only he'll be able to decrypt it. Before we do that, let's make sure we have his public key on our computer...

{% highlight bash %}
$ gpg --list-keys --quiet
/Users/aaronlahey/.gnupg/pubring.gpg
------------------------------------
pub   rsa4096 2018-09-11 [SC] [expires: 2019-09-11]
      B204475FD47FBA3371E656AFF4496594665C6136
uid           [ultimate] Aaron Lahey <alahey@8thlight.com>
sub   rsa4096 2018-09-11 [E] [expires: 2019-09-11]
{% endhighlight %}

Uh oh! The only public key we have on our computer is our own! Since Ben is relatively new to `gpg` maybe we can offer a few suggestions. First, let's figure out his key ID executing the same list command as above on his computer...

{% highlight bash %}
$ gpg --list-keys --quiet
/Users/aaronlahey/.gnupg/pubring.gpg
------------------------------------
pub   rsa2048 2018-09-21 [SC] [expires: 2020-09-20]
      765312983429D55B17AFD25DC4B96D7E4D5256FE
uid           [ultimate] Ben Smith <bsmith@example.com>
sub   rsa2048 2018-09-21 [E] [expires: 2020-09-20]
{% endhighlight %}

This command prints a brief summary of each public key and actually includes more information that necessary. All Ben needs to worry about is the ID which in this case is _765312983429D55B17AFD25DC4B96D7E4D5256FE_. Once we have the ID we can export his public key...

{% highlight bash %}
$ gpg --export 765312983429D55B17AFD25DC4B96D7E4D5256FE
[���.=
V���۝!ƴ��R��0<��8���i]H�����Ʌ����<��U�P9���{�Y�h�H
                                                   �T�Œ�{���T1Ɂ3l/�p��B�l���Ȩ���!cвb·G��|�Ʊ������-�z��y�����Gw�õj
8vÁ��i[���W�7z��$اD���#�bJ�K��LGֹB���r���b��xȆ�E��c�L
�/SHw�!Ben Smit <bsmith@example.com>�>!vS�4)�[��]Ĺm~MRV�[��L	�g


       �
	Ĺm~MRV����+�5
                     s4�ў�Sl�I���]�55cȥ�\xK�(�mJ*�bX;�)��
                                                         EQ&u��Bu�
B���d�L}t�K�*��a�$�S@���|4�<A�e���(��l�1lRj]��f�Id��f���b�ń�KF�;��E
                                                                    Sn���BC�13|���^�Q��AY��ݪaN@��y�
*>2x/$W��h�ᶪpP�&+I�&!vS�4)�[��]Ĺm~MRV�[��L�ȄhI@��g���� �D�H��.%�|��sR^��R���j6׽[���Ԁ��aAF��2�T��^�)�z��wE�WU�5��y�[`~�a�SCm�A{��q}x=�Zg#V��_�w�ޥ���yI�/�6}r�6�
	Ĺm~MRV��t�v���Z��{޷�>#��m�z�N�D��g��Ar�H;D���߷ɭr[�ϘX��ݭ�қL�d~�e�����;C�yd+��X��Scl��;�l�&?�D�K8#ߋ#��忘���W��VZ�c���_�/�󇰂]�n����麏�iknK`��V�����52�ߤ%V���)�v��ů���;u=&����-u�����#G�<��������`e��ι�S]�*��Ѕ�6݌�~E��\�	�[$
{% endhighlight %}

Yikes! What is all this garbage `gpg` printed on his screen? By default, `gpg` exports keys in a binary format and this is how it happens to print at the command line. Let's ask Ben to add the `--armor` flag to export a textual encoding of his key...

{% highlight bash %}
$ gpg --armor --export 765312983429D55B17AFD25DC4B96D7E4D5256FE
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQENBFulgUwBCADELhM9Clalje/bnSHGtL3nUsTnMDynH/I4hPr4aV1Ih8PQy6AI
r4B/yYWlsqDKPMzwVfFQObenxXuqWbhojUgMGJ1U+sWS0HsO0xnst/2tGlQxyYEz
bC+ScL2sEUL3GmzWzuXIqPzn7SESYwbQsmLCtwFH8YPJfIbGsROKurygzuAtq3qJ
oQ4HeZmag6TER3eRbxDMg2oKExyrtF65fj/05cwY2ezxs8io8/KaBUgkk7H/qg+Q
I6NiSvZLselMR9a5QhykxuZyiLzvYqDdE3geyIbiRe2I9Ydj9xcbVEwNOHbDgbrI
EGlbBpab0FeRN3rg8B0k2KdE7dwKxi9TD0h3ABEBAAG0IUNvbGluIEpvbmVzIDx0
cnB0Y29saW5AZ21haWwuY29tPokBVAQTAQgAPhYhBHZTEpg0KdVbF6/SXcS5bX5N
Ulb+BQJbpYFMAhsDBQkDwmcABQsJCAcCBhUKCQgLAgQWAgMBAh4BAheAAAoJEMS5
bX5NUlb+s+sH/isatzUMczSb0Z7LU2yOSeqJBIinXasUNTVjGsil11x4S+y7KOxt
Siq+Ylg76CmrFPMMRVEPJnUEjvZCdf2wCkLb7tBk971MfXSQS/gqBeEQkGHDJMdT
QNThqveQf3w09gY8Qe5lpLClDijI9JIBbNgxbFJqXYv1ZvlJZLucZpQGhbJirsWE
rktGyTvdFLxFC1Nu6BPYzkJDuBcxMwd8j4vXXuBR+ZLuipKjQQ9Zjux+CN2qBWFO
QLu2eZQMuDFs45Cn/XQbDr7COE7WaX12x+w2soZXJSXGuoJ5rtjXbUPcvdN/rAjI
saEjJk8qt5Ki/y4B9jqcpUn9L4U2fXLCNvq5AQ0EW6WBTAEIALGjGh93vjy6JCtn
aAG+SbgIG/vQEKt7dkmV3B98SIpLoJ/B4hiap6Mlp7iqJUbBIB8kHnbDoNMYQBZy
W+aJDVLszkqtP7x0qufmtJhUBJMD6TpRqPuftPrj0+bjMxghcO1lU5uP3t8qn6qe
IOmL50uMyIRoSUCXsymE+sLGIJtE60iCzC4lD5R8kKJzDwBSXrnEUouAr2o2171b
6Mbg1IAWiKhhQUbL9Igyv1SJ5xle+CnUermldxcPRZZXVc01rP95ultgfvlht1ND
baxBe/CL03F9eD3NWmcjVqDUX853mN6lH8/zE/J5DSo+MngvJFflqeNo+Ijhtqpw
UJMmK0kAEQEAAYkBPAQYAQgAJhYhBHZTEpg0KdVbF6/SXcS5bX5NUlb+BQJbpYFM
AhsMBQkDwmcAAAoJEMS5bX5NUlb+5nQH/RJ2jqabWqble9635AU+I837bdl6zk6q
RP+BZ+/sQXIWvUg7ROjKyd+3HMmtcluiz5hYkN/drc7Sm0wbm6JkfgCaZarAhJzr
O0O7eWQr/9gEWPEejlMCY2ydvDv9pLZsFQbiJj+dRIZLOCPfiyOO/Jnlv5i2/+VX
DrShVlr5Y5MXmudf94OILxeb84ewgl3Abo71/vjpuo+9aWtuH0tgjBy+Vsvm7dVM
COSFNTL936QlVq+JnCmsdhquhMWvr56vO3U9JpXS8B/nLXWwBZ0AsYLbI0eaPI7A
mLrhD/HR1WBlhO/OufNTXY8q8oX0t9CF8KY23YzYfkW29lyOEgmTDls=
=APJB
-----END PGP PUBLIC KEY BLOCK-----
{% endhighlight %}

Great! All that's left is to have Ben send us this key and we can import it into our own local copy of `gpg`. Remeber, since this is Ben's *public* key he doesn't need to be at all concerned about security when handling it. He could email it, send it over Slack, upload it to a [public key server](https://pgp.mit.edu/), post it on his personal website, whatever!

For our purposes, however, we need to give a moment of thought to the security implications of our particular mode of communication. If we're chatting on Slack, am I sure it's him at his computer? Am I sure his Slack account wasn't compromised and I'm actually receiving a public key for someone else? If it is him, and he sends the key using email or Slack, are we positive that someone at Google or Slack hasn't intercepted the message and injected their own public key? Likely none of those things are true, but depending on the information you're exchanging and how likely you are to be targeted, it's probably wise to have a relative level of skepticism.

Let's assume we are 100% sure this public key is Ben's. Go ahead and save this key to a file called `ben.asc` and import it...

{% highlight bash %}
$ gpg --import ben.asc
gpg: key C4B96D7E4D5256FE: public key "Ben Smith <bsmith@example.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
{% endhighlight %}
####

...and ensure it's there...

{% highlight bash %}
$ gpg --quiet --list-keys
/Users/aaronlahey/.gnupg/pubring.gpg
------------------------------------
pub   rsa4096 2018-09-11 [SC] [expires: 2019-09-11]
      B204475FD47FBA3371E656AFF4496594665C6136
uid           [ultimate] Aaron Lahey <alahey@8thlight.com>
sub   rsa4096 2018-09-11 [E] [expires: 2019-09-11]

pub   rsa2048 2018-09-21 [SC] [expires: 2020-09-20]
      765312983429D55B17AFD25DC4B96D7E4D5256FE
uid           [ unknown] Ben Smith <bsmith@example.com>
sub   rsa2048 2018-09-21 [E] [expires: 2020-09-20]
{% endhighlight %}

## Trusting a Public Key
Nice! We can now use this public key to encrypt a message to Ben! To make things a bit easier we can refer to this key using Ben's email address. Also, let's not forget the `--armor` flag. Just like exporting a key, encrypting a message will output binary data by default. Here's the command...

{% highlight bash %}
$ cat password.txt | gpg --encrypt --armor --quiet --recipient bsmith@example.com
gpg: FCBCAAE5AA521807: There is no assurance this key belongs to the named user
sub  rsa2048/FCBCAAE5AA521807 2018-09-21 Ben Smith <bsmith@example.com>
 Primary key fingerprint: 7653 1298 3429 D55B 17AF  D25D C4B9 6D7E 4D52 56FE
      Subkey fingerprint: 3D74 62B3 DC49 9C07 E208  A344 FCBC AAE5 AA52 1807

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N)
{% endhighlight %}

Hmmmm. It seems that even though we're sure the key is authentic, `gpg` has its doubts. To inform `gpg` that this is a public key we trust we need to sign it (please be sure to [validate this key's authenticity](https://futureboy.us/pgp.html#ProcedureForVerification) first). Remember from earlier, when we sign something, we're actually encrypting it with _our secret key_. To sign a key we need open the interactive `gpg` prompt...

{% highlight bash %}
$ gpg --edit-key bsmith@example.com
gpg (GnuPG) 2.2.8; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa2048/C4B96D7E4D5256FE
     created: 2018-09-21  expires: 2020-09-20  usage: SC
     trust: unknown       validity: full
sub  rsa2048/FCBCAAE5AA521807
     created: 2018-09-21  expires: 2020-09-20  usage: E
[  full  ] (1). Ben Smith <bsmith@example.com>

gpg>
{% endhighlight %}

At this prompt, type `sign` and confirm our choice...

{% highlight bash %}
gpg> sign
gpg: using "B204475FD47FBA3371E656AFF4496594665C6136" as default secret key for signing

pub  rsa2048/C4B96D7E4D5256FE
     created: 2018-09-21  expires: 2020-09-20  usage: SC
     trust: unknown       validity: unknown
 Primary key fingerprint: 7653 1298 3429 D55B 17AF  D25D C4B9 6D7E 4D52 56FE

     Ben Smith <bsmith@example.com>

This key is due to expire on 2020-09-20.
Are you sure that you want to sign this key with your
key "Aaron Lahey <alahey@8thlight.com>" (B204475FD47FBA33)

Really sign? (y/N) y

gpg>
{% endhighlight %}

Once that's done we can save the key by typing `save` and pressing enter.

{% highlight bash %}
gpg> save
{% endhighlight %}

## Encrypting and Decrypting Our Secret
Now that we've instructed `gpg` to trust Ben's public key, let's give encrypting that password another try...

{% highlight bash %}
$ cat password.txt | gpg --encrypt --armor --quiet --recipient bsmith@example.com
-----BEGIN PGP MESSAGE-----

hQEMA/y8quWqUhgHAQgAjtfuu2yWYEBwNO/BmZp/uizy4z7BpxJK868aMaWRbYcc
3c4k76ofs8OWSddoGVBSt1Os2ENhAZspXGJeb+La+Y8Qz8UA9hQ5ciRs3ZLIRfLC
37SjP/73UwGometSSse9UigtN1TfR0uTo4JitY9oxmoPKVCc/+foSIo9soH9V68q
lhyYfkhAEAW+DfYKFYkhBWcwqD2wNtpkwmOwNsPQOcnXEFFuRntVc78a+DtyCQd5
Usz7A1Hy21ff00LPvrlbhs1YQl/2qqYQ6HRt0Z0r9K4qVNOLQlAWa5MjkXwkSU7q
KOGk1iSQ+Pw9Xm3qM0tSZ+/gjomMTI8npsF800Yq4tJKARUhzqdjjZyg4Chmbeqh
hIFSfGUj+3+DEF3DIWJMKnti98bzqBTqdwzh3t9kX2/Revd6rFz+4/B7kUj+srVj
JcT7K6/PWKaOtmQ=
=nkZB
-----END PGP MESSAGE-----
{% endhighlight %}

Success! We can send this to Ben in an email or via a Slack message and nobody except him will be able to decipher it! Unfortunately, it looks like Ben is struggling to decrypt this message. We can help him out by telling him to run the following command...

{% highlight bash %}
cat encrypted-password.txt | gpg --quiet --decrypt
secretpassword
{% endhighlight %}

## Conclusion
This might seem like a lot of work just to send a message to someone, but here's the good news: If we want to send any more encrypted messages to Ben the majority of the work is already done! We only need to repeat the steps from the previous section.

It's also worth looking at tools like [Keybase](https://keybase.io/). Keybase aims to make this process much more user-friendly. It also aims to solve the public key trust problem mentioned above. You can upload public keys to Kebase and then correlate your Keybase identity with your identity on various social platforms such as [Twitter](https://twitter.com/aaronlahey/status/1041155730869571584) or [Github](https://gist.github.com/aaronlahey/78a16c473d9c884812406e315c124c76).

If you're interested in this stuff, this [GPG Tutorial](https://futureboy.us/pgp.html) is an excellent resource and will guide you through more advanced topics such as [generating a revocation key](https://futureboy.us/pgp.html#RevocationKey).

Keeping our company and our client's secrets private is important! So remember, regardless of whether you use `gpg` directly or you move to a tool like Keybase: _dance like no one's watching and encrypt like everyone is_.
