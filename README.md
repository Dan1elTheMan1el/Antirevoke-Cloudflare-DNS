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

At this point we're almost done setting up the DNS, but you have the option of setting up an adblocker. Since adding thousands of hosts by hand would be just a little tedious, here are [in-depth instructions](https://github.com/mrrfv/cloudflare-gateway-pihole-scripts) on a script that automates the whole process (runs very quickly, and includes automatic updates to host list). Check it out!

Whether you setup the ad-blocker or not, you will need to note down your **Account ID** and an **API Token**.
- **Account ID**:
    - Should be visible in the site's URL: `https://one.dash.cloudflare.com/{ACCOUNT_ID}/*`
    - Note it down
- **API Token**:
    - Head to your [profile's API Tokens](https://dash.cloudflare.com/profile/api-tokens)
    - Create Token -> Custom Token
    - Token name: (Anything)
    - Permissions: `Account`, `Zero Trust`, `Edit`
    - Finish token creation and note it down

Final steps! Head back to the Zero Trust dashboard, and in the sidebar navigate to **Networks -> Resolvers & Proxies**. Add your Default Location, and setup:
- Select DNS endpoints:
    - DNS over HTTPS (DoH) -> ON
    - Note down URL (Should be `https://{SOMETHING}.cloudflare-gateway.com/dns-query`)

And we're done setting up the backend!

### 2. DNS Profile

For this step, you'll need the DoH URL from the end of step 1. Start with opening up your favorite Terminal and running:

```bash
nslookup {SOMETHING}.cloudflare-gateway.com
```
(Make sure not to include the `https://` or `/dns-query`)
You should see an output like:
```bash
Server:		{not important}
Address:	{not important}

Non-authoritative answer:
Name:	{SOMETHING}.cloudflare-gateway.com
Address: {IMPORTANT IP}
Name:	{SOMETHING}.cloudflare-gateway.com
Address: {IMPORTANT IP}
```
Note down those `IMPORTANT IP`s at the end. Next, use [this Shortcut](https://www.icloud.com/shortcuts/115adcb54cbf4c69a5fb246c567fce46) to generate and install a configuration profile by entering the full DoH URL and then a line separated list of IP Addresses that came up from the `nslookup`. Install the configuration profile in **Settings -> General -> Profile & Device Management**.
> [!NOTE]
> Your DNS will automatically switch to this one once you install, so make sure all DNS policies are enabled on your Cloudflare Zero Trust dashboard!

### 3. Toggle PPQ Shortcut
Have your Account ID and API Token handy. Install the [Toggle PPQ Shortcut](https://www.icloud.com/shortcuts/bf1844c12ade478e944c1f4536e54527), and follow the setup instructions. If you leave something blank or make a mistake, you can edit the first action's "Values" later.

That's it! You can share the shortcut to add it to your home screen.

## Usage

Once you've set it up, there's really no maintenance you have to do. Just like any other DNS method, keep the DNS on at all times, and don't use VPNs unless they work with the DNS config.

When you want to install an app, run the toggle shortcut to enable `ppq.apple.com` briefly, open the app once, then run the toggle shortcut again. Note that it turns airplane mode on / wifi off for a few seconds to refresh your DNS, so don't run it while installing anything.

That's all! If this tutorial or my shortcuts were helpful, leave a star on my repo or [Buy me a coffee](https://ko-fi.com/danielthemaniel).

Enjoy :)