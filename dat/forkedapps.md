# Forked DWeb Apps
A Forked Web App is a special kind of p2p app on Beaker that works by having users fork their own copy of an app (which Beaker let's you do easily) in order to populate it with data from their own User Dat. Unlike most web apps on the modern web, DWeb Apps don't interact with a server, instead they pull data either from the user's User Dat or other Dats on the network.

Forked Apps allows user to have much more freedom over how they use the web. Instead of users all navigating to a single domain in order to use an app, they can fork a version of it locally and use it as they regularly would except now the user is not chained to a platform. Thanks to libraries like WebDB and protocols like Dat, users can still be connected to other users. If something about an app displeases a user, they can now modify it on their own end. Now users won't be forced to accept an update they don't want. Another added benefit from forking apps of course includes better privacy. In fact, since your info is yours to keep, forked apps are private by default. The only info being exposed to the outside world is your public facing user info such as username and bio. This is a major step for making the web a safer and better place.

With the introduction of forked apps, we begin to see how apps are no longer do anything important like hosting our data and making sure it's secure, they're merely experiences we can port our information to. Think of it as an add-on or skin to your social media experience. This goes back to a vision for Mimo that we've had since we started which was an emphasis on **experiences, not services**.

[Rotonde](https://louis.center/p2p-social-networking/) is a close example of a forkable app, however they do not separate the user's profile info from the app itself. What this means is that the user can't take their user info to another app.

#### Technical Details
[WebDB](https://github.com/beakerbrowser/webdb) is a database that reads and writes records on DatArchives. With WebDB, you can index archives and prep them for querying. WebDB is the backbone of any DWeb App on Dat/Beaker Browser.

WebDB will look at the files in your archive and index them so that when you load a page on a DWeb App, WebDB will be ready to fetch the data you need and display it on the page.
