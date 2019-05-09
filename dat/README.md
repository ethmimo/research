# P2P Identity on Dat: A Tentative Framework

Here's where we outline our vision and research on using Dat to power a p2p identity layer.

## User Profiles
User profiles are a collection of files inside a `profile/` folder inside a [DatArchive](https://beakerbrowser.com/docs/apis/dat). User info is held in a `profile.json` file and following lists are held in a `following.json` file.

It's important that user data be kept separate from the app itself because we want to allow users to seamlessly switch between apps without losing their data from one app to another.

## DWeb Apps
Distributed Web Apps (DWeb Apps) are web apps (which are basically some files) hosted on Dat instead of http, so anyone on the p2p dat network can retrieve the files for the app as long as someone is seeding (hosting) them. Instead of user info being siloed away, held on remote servers, they are hosted by people on the network

When a user visits a DWeb App, instead of a request going off to a remote server, the app is instead populated with data from the visitor's own user profile Dat. This means the app reads your files (this is all done locally btw, the app cannot send your data anywhere else) and fills the app with whatever data it needs. So to show your personal Twitter feed for example, the app will read your DatArchive for a `posts/` folder and display the files in that folder on your feed.

This, combined with forking Dats into your own library which enables [Forked Apps](forkedapps.md), allows user to have much more freedom over how they use the web. Instead of users all navigating to a single domain in order to use an app, they can fork a version of it locally and use it as they regularly would except now the user is not chained to a platform. Thanks to libraries like WebDB and protocols like Dat, users can still be connected to other users across the globe, even though users are now interacting with the web in an "isolated" manner. If something about an app displeases a user, they can now modify it on their own end. Now users won't be forced to accept an update they don't want. Another added benefit from forking apps of course includes better privacy. In fact, since your info is yours to keep, forked apps are private by default. The only info being exposed to the outside world is your public facing user info such as username and bio. This is a major step for making the web a safer and better place.

With the introduction of forked apps, we begin to see how apps are no longer do anything important like hosting our data and making sure it's secure, they're merely experiences we can port our information to. Think of it as an add-on or skin to your social media experience. This goes back to a vision for Mimo that we've had since we started which was an emphasis on **experiences, not services**.
