---
layout: post
title: Send a fax from the command line with Python and Phaxio
commentIssueId: 5
tags: Python, Fax, Send Fax, CLI Fax, Phaxio
---

If you want to send a simple fax quickly, cheaply, and painlessly, Phaxio and Python make a nice combo. Below is a litte script that I wrote, based on this [Ruby script by Pete Keen](https://www.petekeen.net/command-line-faxing) that is slightly out of date. There are Phaxios Python libraries, but I ran into a couple issues, and this seems to be the most brain-dead simple solution. Pros: No external dependencies. Cons: It uses the `shell=True` parameter for `subprocess.call`, but that shouldn't be an issue since you're only using this to send a quick fax at 2 AM and you don't want to pay UPS/FedEx/whomever too much money for that privilege tomorrow, right?

Note that I'm not affiliated with Phaxio in any way, it just happens to be late, I needed to send a fax, and they checked all the right boxes. I stumbled on Phaxio, but for someone just wanting to send a quick fax once every year or so, it's great. Pricing is about $0.07 a page (and I received $1.00 account credit just for signing up at the time of writing this), so it's perfect for my use case.

## Setup

1. Sign up for an account with [Phaxio: https://www.phaxio.com/](https://www.phaxio.com/)
2. Get your API Keys: [Phaxio API Credentials](https://console.phaxio.com/api_credentials)
3. Put them into the script below (You can also use the Test keys to make sure this works before trying too.)
4. Run the script, for example, if I saved the script to `fax.py`, I'm sending to Tommy Tutone, and my file to send is `letter.pdf`, I would use the following: `./fax.py +15558675309 /path/to/letter.pdf`

```python
#!/usr/bin/env python3
from subprocess import call
import sys

if len(sys.argv) <= 2:
	print("Usage: send_fax NUMBER FILENAME...")
	exit(-1)

number = sys.argv[1]   
api_key = 'put_api_key_here'
api_secret = 'put_api_secret_here'

command_args = [
  "curl",
  "https://api.phaxio.com/v2/faxes",
  "-u '{}:{}'".format(api_key, api_secret),
  "-F 'to={}'".format(number)
]

for file in sys.argv[2:]:
	command_args.append("-F 'file=@{}'".format(file))

call(' '.join(command_args), shell=True)
```

The script can be found in this GitHub Gist here: [https://gist.github.com/seanlane/67504bf39696de8c0bc88ad89844f9df](https://gist.github.com/seanlane/67504bf39696de8c0bc88ad89844f9df)

Feel free to fork it and suggest improvements.