title = "delilah: GitHub status updates in Slack"
date = 2020-01-30
%%%

[Repo](https://github.com/evuez/delilah)

A [lot](https://www.githubstatus.com/) [of](https://status.circleci.com/) [company](https://status.rollbar.com/) use [Statuspage](https://www.statuspage.io/) to build status pages for their services.
You can subscribe to status updates via email or text message, but there's no integration with Slack.

At work we rely on GitHub for our deployments, and trust me, you don't want to start a deployment when GitHub is down.
[delilah](https://github.com/evuez/delilah) was built to help with this: we get status updates from GitHub directly in Slack and this makes it easier to catch these before starting a deployment.

It can be configured with any service using Statuspage and will post a message in Slack whenever they update their status page.

It's a pretty simple [Scotty](https://github.com/scotty-web/scotty) app -- it's basically forwarding requests from Statuspage to Slack -- but it was really fun to write, and proved to be quite useful!

--

P.S.: if you're wondering about the name, Delilah is a character in [Firewatch](https://firewatch.fandom.com/wiki/Delilah).
