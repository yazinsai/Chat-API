# Updates to the application

## Registration

I realized that the version of the library on Composer was older than that on the repo. 
So, I did two things:

1. `git clone https://github.com/mgp25/Chat-API` into the `vendor/whatsapp/` directory
2. Registered the `Registration` class by editing `composer/autoload_classmap.php`:
   `'Registration' => $vendorDir . '/whatsapp/chat-api/src/Registration.php'`
   There was an error when first starting because the "Registration" class was not, 
   well, registered with the autoloader.

**Old Version**

Now, when running it I got the error `old_version`. Progress! Ok, so now it's clear that
we need to update a version somewhere. The obvious place was `src/Constants.php` where I
updated it to the latest WhatsApp version from my installed app (2.19.81). Ran it, and 
now I get the error `bad_token`. Ok, it's not that simple. The APK appears to hardcode
some values that need to be updated in `src/token.php`. Looks like I'll need to reverse
engineer the app, to at least pull out the string constants we'll need.

**Bad Token**

To get this going, I'll need to decompile the APK, and run it through DEX2jar. We then run
the output through [JD-GUI] or [Procyon] to view the Java code. Search for something that
resembles a token and we're done

Let's do it:

1. Pull the APK from your phone; I used [MyAppSharer]
2. Run JD-GUI on the APK
3. View the resulting JAR using JD-GUI or Procyon
4. Find the relevant string values

I wasn't able to find the strings that I'm looking for, so I changed the strategy abit.
I looked up the old APK for the last version referenced in the `token.php` file; it was
[v2.16.148].

I then reversed this APK and tried to find the references. Nothing. Huh? How come? I
then tried to Base64decode the string, and then search for that. I found some printable
characters that I then searched by "California1". That worked -- I found many references
though, probably because this is related to an SSL certificate (for some sort of Google
signed verification). I searched for more specific strings (e.g. Brian Acton) but got
nothing.

Other attempts made:
1. Searched for MIID (the starting sequence of characters); no result
2. Searched for the $key2 value
   - Base64 encoded value search "14w/wF67XBf2vTc"; nothing
   - Base64 decoded value using `(Base64.strict_decode64 "14w/wF67XBf2vTc+qALwKQ==")`
     which results in `"\xD7\x8C?\xC0^\xBB\\\x17\xF6\xBD7>\xA8\x02\xF0)"`
      1. Base64 decoded the string (using UTF8) and searched in ASCII; nothing
      2. Searched for the Base64 decoded string in Hex; nothing
      3. Searched for the Base64 decoded string in Binrary; nothing

Read through the [WA RE documentation] and found out the following:
- `$classesMd5`: MD5 hash of the 'classes.dex' file (see [ClassesMD5]). For the version
  I was using (`2.19.81`), the value was `"p1J+2B1aBTuUMhoDkoECBA=="`
- `$key2`: the precalculated value listed in the [Documentation - Android] section of
  the RE-Whatsapp repository. I was able to get the key generation script working, and
  publised the code here (Note: It requires the APK file)
  
Now, it's time to test the updated `token.php` file with the changes I've made.

[MyAppSharer]: https://play.google.com/store/apps/details?id=com.yschi.MyAppSharer&hl=en_US
[JD-GUI]: https://java-decompiler.github.io/
[Procyon]: https://bitbucket.org/mstrobel/procyon/downloads/
[v2.16.148]: https://www.apkmirrordownload.com/apk/whatsapp-messenger-2-16-148-451238-apk/download-apk/
[ClassesMD5]: https://github.com/mgp25/classesMD5-64
[Documentation - Android]: https://github.com/mgp25/RE-WhatsApp#android-token
