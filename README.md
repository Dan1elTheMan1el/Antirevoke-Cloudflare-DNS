# Guide to using Anti-Revoke DNS with Cloudflare
by DanielTheManiel

## Intro

### What is the "DNS Method"?
The "DNS method" of sideloading on IOS involves using a signer like KSign, ESign, or Feather along with a revoked certificate to sideload apps without a computer, weekly refreshes, or app limits. The DNS is put in place to block Apple's ability to check the status of the certificate, allowing you to use it indefinitely.

### Why Cloudflare?
Although the setup is a little involved, this method allows for:
- No limit on monthly queries
- Data is completely private
- Toggleable PPQ Blocking and adblock using Shortcuts/API
> [!TIP]
> PPQ Blocking will be introduced later - being able to toggle it ensures you can install apps safely without blacklisting your device

The only downside would be that signing up for the **Free** tier requires a credit/debit card or a paypal account linked to your Cloudflare, but as long as you stay on the **Free** tier it should be fine.

That being said, if you are using NextDNS and are not close to its 300,000 monthly query limit, I recommend you stay with it. You can use [my shortcut](https://browse.shortcuty.app/shortcut/2999e365-fae4-4fdc-a488-1d08f7b52221) to toggle PPQ on NextDNS, or follow [surrel14's guide](https://github.com/surrel14/dnsmethod.github.io/blob/main/README.md) for full setup.

## Setup
Get ready!

### 1. Cloudflare Zero Trust
Create a cloudflare account if you don't have one already, then head to the [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com).
> [!NOTE]
> The entire website was super buggy and unusable for me on Firefox. Use Chrome or another browser if you have any issues.

If it's your first time there, it will ask you for:
- Team name
    - (Any)
- Plan
    - (Free)
- Payment method
    - (Won't charge anything)
    - Business type: Personal

Head back to the dashboard, and in the sidebar on the left heat to **Reusable components -> Lists**. You will now configure the list of domains to block. Click the **+ Upload CSV** button, and setup:
- List name: (Anything)
- List type: `Hostnames`
- CSV File: [`blacklist.csv`](https://github.com/Dan1elTheMan1el/Antirevoke-Cloudflare-DNS/releases/download/v0-alpha/blacklist.csv)

Then click create.

Next, in the sidebar navigate to **Traffic policies -> Firewall policies**, then click **+ Add a policy**.
- Policy name: (Anything OTHER than `ppq`)
- Traffic:
    - Add condition
    - Selector: `Domain`
    - Operator: `in list`
    - Value: (List you created previously)
- Action: `Block`

Then click Create policy.

Now we'll make another policy solely for `ppq.apple.com`. This one will be toggled off when installing apps, then toggled back on for regular use without affecting the other domains. **+ Add a policy**:
- Policy name: `ppq` (Name must be exact to use the Shortcut toggle)
- Traffic:
    - Add condition:
    - Selector: `Domain`
    - Operator: `is`
    - Value: `ppq.apple.com`
- Action: `Block`

Then click Create policy.

At this point we're almost done setting up the DNS, but you have the option of setting up an adblocker. Since adding thousands of hosts by hand would be just a little tedious, here are [in-depth instructions](https://github.com/mrrfv/cloudflare-gateway-pihole-scripts)
