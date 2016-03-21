#Design Messenger System#
[Messenger](https://code.facebook.com/posts/820258981365363/building-mobile-first-infrastructure-for-messenger/)
##Client Side##
1. Notification is done by long polling.
* client sends request to server, server holds the connection till data/event is available
* once client receives the data it resends another request right away
1. Push+pull snapshot
* Initial HttpCall to pull snapshot of the chat
* Then just receives notification and add delta (changed messages)
1. Protocol: Http->MQTT(low bandwidth protocol)
1. Message Format: JSON->Thrift 
## Server side ##
1. MessageQueue (Iris) backed up with Disk
* Queue of messaging updates (new messages, message state change)
* Pointers into the queue to indicate last update sent to messenger app/user/disk
* Queue is the write-through cache
* Queue should have some backing store of a week in case failover
  * Older conversation history will be deleted from the queue/backstore and pushed into disk
* Long Disk seek will not impact read

#Comments auto appending#
Auto refresh when new comment shows up for certain post, no need to refresh page
[Original Post](https://www.facebook.com/notes/facebook-engineering/live-commenting-behind-the-scenes/496077348919/)
##Polling Model##
For every page that had comment-able content, the page would periodically send a request to check whether new comments had arrived.
* Frequent polling (less than 5 seconds) will overlead the server
* Need a push model
  * When comments, needs to know all users viewing the same post
    * When user views a content, save some information in local data center on viewing event (user, postId)
    * When user wants to post, it pulls all viewers from all different data centers on who is viewing and send updated comment notifications to all users.
  * Once user receives push notifications, they can pull the latest comments for that feed.

#@Someone functionality#
1. FrontEnd
* Capture keystroke event, detect last@ position and find string after that
* Use the string to get list of suggestions for each user
* Open a span and capture which is selected, auto complete.
2. A new type of notification system.
* Send messge into target Inbox.

#Feed API#
##FB style
Data getAlbums(mediaId, after, before, limit)
->
{
	data :[],
	paging: {
		"cursors": {"after": "foo", "before" : "bar" },
		"prev": "https://graph.facebook.com/mediaId/albums?limit=25&before=foo", //using prev to trace back further
		"next": "https://graph.facebook.com/mediaId/albums?limit=25&after=bar"  //using next to refresh newly added
	}
}
##Twitter Style:
Data getAlbums(mediaId, sinceId, maxId, limit)

"search_metadata": {
  "max_id": 110,
  "since_id": 001,
  "refresh_url": "?since_id=110&q=mediaId", //using since_id for refresh
  "next_results": "?max_id=100&q=php&count=10", //using maxId for pagination
  "count": 10,
  "completed_in": 0.035,
  "since_id_str": "001",
  "query": "php",
  "max_id_str": "110"
}

#TODO More Read, some are old#
https://www.facebook.com/notes/facebook-engineering/scaling-the-messages-application-back-end/10150148835363920/
https://www.facebook.com/note.php?note_id=454991608919
https://www.facebook.com/notes/facebook-engineering/facebook-chat/14218138919/
