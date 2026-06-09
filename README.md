📘 Ring Firewall External Dynamic Lists (EDLs)
This repository provides External Dynamic Lists (EDLs) for Palo Alto Networks firewalls to securely control outbound traffic from Ring Doorbells and Ring Cameras.

Ring devices rely heavily on AWS infrastructure, including:

Core AMAZON service ranges

Global CLOUDFRONT edge networks

Multiple AWS regions (not just the local region)

These EDLs allow you to tightly restrict Ring traffic to only the AWS IP ranges Ring actually uses, while blocking all other outbound destinations.

📂 Files in This Repository
ring-aws.txt
Primary EDL containing IPv4 CIDRs used by Ring devices across:

ap-southeast-2

us-east-1

us-west-2

Includes both:

AMAZON service ranges

CLOUDFRONT service ranges

This is the main list your Palo Alto firewall should reference.

ring-cloudfront.txt
CloudFront‑only IPv4 CIDRs for the same regions.

Use this if you want to separate:

Core AWS services

CloudFront edge nodes

Some administrators prefer separate EDLs for more granular rule control.

⚙️ How These EDLs Are Generated
A GitHub Actions workflow automatically updates both files every 6 hours.

The workflow:

Downloads the official AWS IP ranges JSON
https://ip-ranges.amazonaws.com/ip-ranges.json

Filters the prefixes using jq

Writes the results into ring-aws.txt and ring-cloudfront.txt

Commits changes only when the files have changed

This ensures your Palo Alto firewall always receives fresh, accurate AWS IP ranges.

🧩 jq Filters Used
AMAZON + CLOUDFRONT (primary EDL)
bash
curl -s https://ip-ranges.amazonaws.com/ip-ranges.json \
| jq -r '.prefixes[]
  | select((.service=="AMAZON" or .service=="CLOUDFRONT")
           and (.region=="ap-southeast-2" or .region=="us-east-1" or .region=="us-west-2"))
  | .ip_prefix'
CLOUDFRONT only (secondary EDL)
bash
curl -s https://ip-ranges.amazonaws.com/ip-ranges.json \
| jq -r '.prefixes[]
  | select(.service=="CLOUDFRONT"
           and (.region=="ap-southeast-2" or .region=="us-east-1" or .region=="us-west-2"))
  | .ip_prefix'
🔄 GitHub Actions Automation
The workflow file is located at:

Code
.github/workflows/update-edl.yml
It runs every 6 hours and updates both EDL files automatically.

Workflow Summary
Fetch AWS IP ranges

Filter with jq

Update EDL files

Commit only if changes exist

Push to GitHub

Your Palo Alto firewall will automatically refresh the EDL on its configured polling interval.

🔥 Using These EDLs on Palo Alto Firewalls
1. Create the EDL objects
In Objects → External Dynamic Lists → Add:

Ring-AWS-EDL
Code
https://raw.githubusercontent.com/jonni-star/ring-edl/main/ring-aws.txt
Ring-CloudFront-EDL
Code
https://raw.githubusercontent.com/jonni-star/ring-edl/main/ring-cloudfront.txt
🛡️ Recommended Security Policy Structure
1. Allow Rule
Code
source: <Ring device IP>
destination: [ Ring-AWS-EDL Ring-CloudFront-EDL ]
service: <Ring TCP/UDP service objects>
action: allow
log-start: yes
log-end: yes
2. Deny Rule
Code
source: <Ring device IP>
destination: any
action: deny
log-end: yes
This ensures:

Only AWS + CloudFront IPs are reachable

All other outbound traffic is blocked

Logs clearly show allowed vs denied flows

🧪 Verifying EDL Operation
Check EDL contents:
Code
show external-list Ring-AWS-EDL
show external-list Ring-CloudFront-EDL
Check Ring traffic:
Code
show log traffic | match <Ring IP>
You should see:

Allowed traffic hitting your allow rule

Denied traffic hitting your deny rule (if misconfigured)

📬 Contributions
Pull requests are welcome for:

Additional AWS regions

IPv6 support

TURN/STUN‑specific EDLs

Region‑segmented EDLs
