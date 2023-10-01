# friend2friend-secure-connection
Options to securely connect to friends network for offsite backup

rather than traditional VPN, use Tailscale/tailnet to grant access to individual server ip/port 

Inspiration:
- HA Norge Q&A sessions for knowledge sharing and concept evaluation
- KennethM for techincal know-how on setup,, early testing to get solution actually working :)
- Kim Svendson https://www.facebook.com/groups/217132562182318/permalink/1426676614561234/


concept high level:

I share my unRAID server smb-service through tailscale/tailnet.
On unRAID server different shares for different users. for each user create shared and private shares. private shares have files that shall not be open by server-owner


Concept Detailed:
- for me to have a server that is 24/7 on,, with permanent internet connection and parity protection from drive-failure 
- on that server I create a share and userID/pwd for my friend(s) to upload their "offsite-backups"
- all shares must be drive-fail-protected, mirror, raid or parity. this way data is some kind of protection against drive-failure 
- then I create a secure Tailnet in which I share my server (IP and relevant ports) to my friens(s) which will create tailnet-users for them
  - Tailnet will then create a secure connection between my server and my friends device which is connected to my tailnet with his tailnet-user
- On my server (in tailnet) I will then see my friend tailnet-id (email) to which I can set security measures in my tailnet (ACL)

<img width="1145" alt="image" src="https://github.com/ArveVM/friend2friend-secure-connection/assets/96014323/49533511-ddff-42e8-8d09-0ce0d545e911">


## 1. Setup Tailnet
1. go to https://tailscale.com/
2. create free personal account (using one of their accepted logon-providers (google, github etc)
3. this account will be the tailnet-admin

## 2. Setup/share unraid Server in Tailnet
1. Log into unraid
2. install Tailscale-plugin
3. in tailscale-plugin, log into Tailnet with tailnet-admin-account
4. make sure NetBIOS=OFF in smb-settings
5. setup proper ACL, limit to port 445 if no other services than smb-file-transfer is required
6. share server with friend,, click Share from Machines-tab (on server which you want to share in Tailscale Admin-console and copy link 

## 2. Setup unraid user/shares
1. Log into unraid
2. Create new User
3. Create new share, grant user r/w access to Share
4. inform user of share-name/pwd




'''

	// ACL to enable acces in Tailnet; full access for admin, and limited for ExternalBackup_users (extbck_users)
	{
	// Declare static groups of users. Use autogroups for all users or users with a specific role.
	"groups": {
		"group:xtbck_users": [
			"admin@gmail.com",
			"exbck_xx1@gmail.com",
		],
	},

	// Define the tags which can be applied to devices and by which users. ArveVM; autogroup:admin is referring to tailnet-admin
	"tagOwners": {
		"tag:server-admin": ["autogroup:admin"],
	},

	// Define access control lists for users, groups, autogroups, tags,
	// Tailscale IP addresses, and subnet ranges.
	"acls": [
		// Allow all connections for tailnet-admin
		{
			"action": "accept",
			"src":    ["tag:server-admin"],
			"dst":    ["*:*"],
		},
		{
			"action": "accept",
			"src":    ["group:xtbck_users"],
			"proto":  "tcp",
			"dst":    ["100.100.166.80:82,445"],
		},
	],

	// Define users and devices that can use Tailscale
	// SSH.
	"ssh": [
		// Allow all users to SSH into their own devices in check mode.
		// Comment this section out if you want to define specific restrictions.
		{
			"action": "check",
			"src":    ["autogroup:member"],
			"dst":    ["autogroup:self"],
			"users":  ["autogroup:nonroot", "root"],
		},
	],

	// Test access rules every time they're saved.
	// "tests": [
	//  	{
	//  		"src": "alice@example.com",
	//  		"accept": ["tag:example"],
	//  		"deny": ["100.101.102.103:443"],
	//  	},
	// ],
}

'''


todo:
- script samples for rsync encrypted tar-files,,, or other way of packeting stuff in its simplest way
- guide on setup Duplicacy
- guide on test connection/speed


