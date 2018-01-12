prosody_nextcloud_auth
======================

An [external authenticator](https://modules.prosody.im/mod_auth_external.html) for [prosody](https://prosody.im) that lets you hang your prosody users off of your Nextcloud user database.
When a user tries to log in via XMPP, it bounces and tries to login via Nextcloud's HTTP, and reports back.
You should probably be able to use this with ejabberd too I think but I haven't tested that yet.

## Usage:

Install [mod_auth_external](https://modules.prosody.im/mod_auth_external.html).
It's currently in the community repository so you'll need to copy it
to `/usr/lib/prosody/modules` (or equivalent on your system) manually,
or [install the complete community module repo](https://prosody.im/doc/installing_modules).

This would mean doing something like

```
# cd /usr/lib/prosody
# hg clone https://hg.prosody.im/prosody-modules/ prosody-modules

# then insert this into /etc/prosody/prosody.cfg.lua:
plugin_paths = {"/usr/lib/prosody/community-modules"}                                                                                                                                             ```

That done, install this script to your system.
```
cp nextcloud_auth /usr/local/bin/
chmod +x /usr/local/bin/nextcloud_auth

# Make sure it runs; it should print 1 or 0 depending on if the password is correct or not. 
echo "auth:test:example.com:password" | nextcloud_auth https://yournextcloudsite.com
```

Finally, configure your prosody to use the new auth system:

```
authentication = "external"
external_auth_command = "/usr/local/bin/nextcloud_auth https://yournextcloudsite.com"
```

Restart prosody, and try logging in. If you have a user "alice" with password "carrots" on https://yournextcloudsite.com, then she should be able to login as alice@yourxmppsite.com with the same password, and *should not* be able to login with a different password. Please make sure to test both.
