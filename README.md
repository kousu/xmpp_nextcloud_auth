prosody_nextcloud_auth
======================

An [external authenticator](https://modules.prosody.im/mod_auth_external.html) for [prosody](https://prosody.im) that lets you hang your prosody users off of your Nextcloud user database.
When a user tries to log in via XMPP, it bounces and tries to login via Nextcloud's HTTP, and reports back.

You might be able to use this with ejabberd too I think but I haven't tested that yet.

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

Finally, configure your prosody to use the new auth system. Ideally that would look like:

```
authentication = "external"
external_auth_command = "/usr/local/bin/nextcloud_auth https://yournextcloudsite.com"
```

Unfortunately, currently [mod_auth_external](https://hg.prosody.im/prosody-modules/file/tip/mod_auth_external/mod_auth_external.lua)
assumes `external_auth_command` is just your program; it [cannot take arguments](https://prosody.im/issues/841).
There's two solutions: 1) apply the patch in [#841](https://prosody.im/issues/841) and go advocate for it to be applied globally.
2) Write a wrapper script:

```
mkdir -p /opt
cat > /opt/nextcloud_auth_yournextcloudsite.com <<EOF
#!/bin/sh
nextcloud_auth https://yournextcloudsite.com
EOF
chmod +x /opt/nextcloud_auth_*

# Then, edit prosody.cfg.lua:
external_auth_command = "/opt/nextcloud_auth_yournextcloudsite.com"
```

Restart prosody, and try logging in. If you have a user "alice" with password "carrots" on https://yournextcloudsite.com, then she should be able to login as alice@yourxmppsite.com with the same password, and *should not* be able to login with a different password. Please make sure to test both.

## Suggestions

If you're building a server where you're attaching chat accounts to "personal cloud" accounts,
you're probably building a project-focused server. In that case you might find
[mod_roster_allinall](https://modules.prosody.im/mod_roster_allinall.html) useful: it bootstraps rosters by putting everyone online in everyone elses' roster automatically.
Similarly, to bootstrap your chatrooms, look at [mod_default_bookmarks](https://modules.prosody.im/mod_default_bookmarks.html).

```
modules = {
  ...
  "roster_allinall";
  "default_bookmarks";
  ...
}

...

default_bookmarks = {
    ...
    "general@conference.yourserver.com";
    ...
};
```

You also might want to disable federation, with

```
modules_disabled = {
  ...
  "s2s";
  ...
}
```

## Alternatives

The more stable, expandable, and generally saner design would be to use [SASL](https://prosody.im/doc/cyrus_sasl), probably with LDAP,
and configure NextCloud to also use LDAP. Or what about making email authoritive with [mod_auth_dovecot](https://modules.prosody.im/mod_auth_dovecot.html)?

`nextcloud_auth` is *not* built for 1000+ user installs. It *will not* scale.