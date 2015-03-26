#!/usr/bin/env run-md-bash
# Disabling SelfControl using `pfctl`

So [SelfControl](https://github.com/SelfControlApp/selfcontrol) is an awesome application,
and helps me remove the question entirely about whether it's a good idea to check my facebook
newsfeed.

Once enabled however, it can become a pain to undo. Blocking facebook for a couple of hours may
seem like a great plan, but when you realise that you can no longer use a facebook portal to login
to spotify then maybe it's a net drop in productivity.

Disabling SelfControl used to be a pretty standard procedure using `ipfw` in pre-yosemite builds,
but now that `ipfw` has been deprecated, SelfControl has changed it's blocking methods.

The app works on two levels...

1. Appending content to `/etc/hosts` that rewires any blocked addresses to localhost
2. Configuring firewall rules that disable access to the blocked domains

The first can be tackled by simply removing the content in `/etc/hosts`.

```bash

echo -n "First lets remove the helper script that reapplies rules periodically..."
sudo rm -f /Library/PrivilegedHelperTools/org.eyebeam.SelfControl
echo " done!"

echo -n "Do you wish to remove /etc/hosts config? [yn] "
read yn

if [ "${yn}" = 'y' ];
then
  echo -n "Removing hosts config..."
  # Remove all lines between # BEGIN SELF, # END SELF
  sudo sed -i '/# BEGIN SELFCONTROL/,/# END SELFCONTROL/d' /etc/hosts \
    && echo " done!"
fi

```

The second is more complex, and much has changed since the switch to `pfctl`.

```bash

echo -e """
Lets inspect the rules currently loaded into pfctl...\n"""

# -s recursively print all anchors
# -r reverse DNS lookup for states when displaying
sudo pfctl -sr 2>&1 | sed 's/^/    /g'

echo -en "\nThere should be an anchor named org.eyebeam? [yn] "
read yn

if [ "${yn}" = 'n' ];
then
  echo "No? Then you probably don't have SelfControl running. Exiting!"
  exit 0
fi

echo -e """\n
eyebeam.org is the name under which SelfControl runs.
It has created and configured this anchor."""


```

Anchors contain a set of rules that can control packet forwarding. We
can remove the rules from the anchor, however...

```bash

echo -e """\n
Listing the rules under eyebeam.org...\n"""

# -a selects anchor on which to operate
# -s displays all rules under selected anchor
sudo pfctl -a org.eyebeam -s rules 2>&1 | sed 's/^/    /g'

echo -e """\n
If SelfControl is currently active, then you should be able to see
roughly one rule per blocked domain.

We can now flush these rules...\n"""

# -F flushes all rules under selected anchor
sudo pfctl -a org.eyebeam -F rules 2>&1 | sed 's/^/    /g'

echo -e """\n
Hopefully when we now list the rules, we won't see any!\n"""

sudo pfctl -a org.eyebeam -s rules 2>&1 | sed 's/^/    /g'

echo -e """\n
Simple!"""



