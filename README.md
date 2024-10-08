# discord-server-guidelines
# Discord Server Guidelines
I compiled a list of attacks I've seen over the years on crypto discords I've managed or that I was part of, along with a list of solutions to them that I've implemented on defilama's discord. The goal of this repo is to serve as a compendium of knowledge so we can prevent more discord hacks by sharing all that iv'e learned in these years.

I've spent a lot of time trying different bots, adjusting them, evaluating them... with this you can skip all and just get to the final solution directly.

## Server settings
- Disable permissions to create public and private threads for everyone. Scammers use these to scam users while making it harder to spot them since threads are not visible to everyone by default.
	- Directions: right click on your server icon > Server Settings > Roles > create a new role > search for "public" in "Permissions" tab > disable "Create Public Threads" > under "Manage Members" tab hit "Add Members" button > select all > save > done.
- Disable permission to tag @everyone
	- Directions: right click on your server icon > Server Settings > Roles > create a new role > search for “mention” in "Permissions" tab > disable “Mention @everyone, @here, and all Roles” > under "Manage Members" tab hit "Add Members" button > select all > save > done.
- Go through all general roles and make sure that on "Display" tab, "Allow anyone to @mention this role" is turned off
	- Directions: right click on your server icon > Server Settings > Roles > examine.

- Set Overview > Default Notification Settings to "Only @mentions", this will stop your users from getting a notification for every new message

- Remove all admin-like permissions, like manage channels... Pay special attention to ensuring that "Manage Webhooks" permissions is turned off, since that's how many big NFT discords were hacked before
- Make sure that for your announcement channels members can only view channel and view channel history, they cant send messages or create threads
- Go to Moderation > Safety setup and enable all the stuff that makes sense for you

## Bots
There's a plethora of third party discord bots that claim to help with moderation, spam, user captchas... I generally don't like these because:
- If they ever get hacked your server will get hacked too, and bots can use their elevated permissions to do whatever they want. This usually leads to user losses when fake airdrop/mint links are shared.
- Usually these bots request admin permissions, and this means that they have access to all channels, even team-only ones. This means that all your internal messages could be leaked to the bot owner, software vendors for the bot or a potential hacker, which is especially bad when info discussed could move your token's price.
- Most of the big bots use patterns that train users to follow behaviour that could be used to scam them.
  - MEE6 sends a DM to users that join your server and requests that they click a link to go outside discord and complete tasks like fill captchas. It's difficult to verify that the user DMing you is the actual bot, so this opens to door for scammers to impersonate the bot and send a DM to a page that will drain the user's wallet. Since users are trained in this pattern before its more likely they'll fall for this
  - Captcha bot tells users to go to an external page where they have to give the bot permissions to their accounts and then fill a captcha. This pattern has been abused by scammers to make bots that impersonate this bot and instead ask for extra permissions that then allow the scammer to take control over the discord account and use it to send scam links to other people. Again, because this is a pattern that users have seen before in your server, they're more likely to fall for this scam.
- A lot of these anti-spam bots work by having new users join the server and go into a "waiting room", then after doing a captcha users are granted a new role that gives them access to the rest of the server. This system however doesn't stop a whole class of spam bots that work by DMing your users individually and trying to scam them in DMs by impersonating the real admins/support of the server, since these bots just need to join the server to DM, they dont need the role and thus don't need to solve captcha. It's possible to make it harder for them by having new users join a channel where they cant see other server members on the right panel, and thus they need to captcha in order to get the list of who to spam, however this approach is already being defeated by bots who have a single bot account go through captcha, get the member list, and then other bot accounts (that haven't solved the captcha) use that list to spam everyone.

 To solve all these issues I've been using a system that's extremely permission-minimized and which is based on these 2 bots:
- Captcha joins: To gate new users I use [f1rewall](https://github.com/0xngmi/f1rewall), this bot simply creates a new webpage where users are asked to solve a captcha and, upon completion, bot generates a single-invite link to the server and sends it to the user. You can test it by going to https://discord.defillama.com/. This works well because:
  - Bot only requires permission to create new invite links, so even if it got hacked all it could do is simply invite spam accounts into server. Bot cannot leak any messages, it can't send scam links...
  - It provides very simple and good UX, user is just shown a captcha and then joins in a straightforward process. There's no need for the user to figure out how things work in this server or search all channels for a way to get verified, it's just simpler.
  - It solves the issue of DM spam bots not having to solve captcha to spam users. You'll still get spam bots but now they all need to solve the captcha, which significantly raises the entry barrier.
  - Doesn't teach user any bad patterns that can rekt them later, users just go to a page which they can easily verify by looking at the domain. No matter what bot you use, user always needs to get the join url correctly, so with other bots there's multiple steps where things can go wrong or where users need to verify things, whereas with this bot the first step is shared but all other steps are removed, so it's strictly better.
  - For users it's much easier to verify the first join link if its like discord.defillama.com vs discord.gg/v42ha2
  - You have full control of url used for joining, so if you later decide to change the method used for joining, or decide to move out of discord, you can simply change the page to link somewhere else (like tg) and all the links you've left scattered before will keep working. If you've been linking discord.gg/v42ha2 instead then you can't make that switch, it's much less flexible
- Discord's native safety bot. Discord has a bot which you can access at Server Settings > Moderation > Safety Setup > AutoMod
  - Enable "Block Custom Words", and on the "Use regex patterns" field introduce the following two expressions separated by a new line: `.*\[.*\]\(.*\).*`, `.*discord\.gg.*`. The first expression will ban messages that try to abuse markdown to mask scam urls and second expression will ban all discord links, i've found that these 2 rules cover almost all messages from scammers/spammers.
  - Enable "Block Words in Member Profile Names" and input your project name and "support" as the list of banned words, eg: `defillama, support`
  - Enable all other options
  - For every section, in "Choose a response", make the bot block the message, time out the user for 5 mins and send an alert to a channel that is only visible to moderators. Moderators can then review the messages received there and ban the user before the timeout expires.
  - In my experience, the following rules will cover almost all the spam/scams while having an extremely low false positive rate, and since it does so using the native bot you don't need to introduce new security assumptions or potential leaks.

## Misc
- Create a separate channel for hi/gm messages so these don't pollute the other channels
- I like to create a new channel for which only admins have access and set the user welcome messages to be sent there. This is useful because you remove the spam from join messages (just mute the channel) and you keep a log of joins that can be used later when banning scammers, since they usually join in blocks.
- Never use a custom join link (eg: discord.gg/myproject), even if its available for you. The problem with these is that if your discord server tier is ever downgraded, you'll lose the link and scammers will be able to register it to their scam servers, then they just need to find some old tweet/forum post/tg message where you're telling someone to join your discord with that link and they can use that legitimate message to trick your users into joining their scam discord instead, where they'll be scammed.
- Make sure everyone with moderator-level access has 2FA set up for their discord account.
- Be careful when deleting an invite link that has been widely linked to, if you do so scammers can then set up a vanity invite link that mirrors your invite link to scam users into joining their discord. vanity urls are usually stuff like discord.gg/myproject, but they can be abused to be set to stuff like discord.gg/v42ha2, to mirror some normal invite link that no longer exists.
- Discord's API allows anyone to query the name of all channels in a discord server, even channels that are not visible to the user calling the API. This means that if you create a channel like #coinbase-listing, an attacker could find out that this channel exists and front-run the listing using insider info, a competitor could find out about your upcoming products... Discord is aware of this issue and has acknowledged, but has no plans to fix it, so you need to be aware that this exsts. Solution is to either be careful with channel names or use another discord server for just team.

## Spear-phishing
There are some spear-phishing attacks that target discord admins or team members specifically in order to compromise the discord server. An example of such attacks involves a scammer claiming to be a journalist for a semi-big publication that says they want to do an interview with you. There are a few variations of this:
- Sometimes they ask you to go on a call with them for which you need to download some special software, when you do so you en dup installing a virus on your computer that compromises it
- Another is that they'll ask you to join a discord (saying you can verify their identity there or some other excuse), and then their captcha bot for new joins will ask you to give it permissions to your account. As soon as you grant it permissions it will take over your account and send scam links on your own discord server, trying to scam your users.
