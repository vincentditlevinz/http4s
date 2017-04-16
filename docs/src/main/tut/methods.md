---
menu: methods
weight: 115
title: HTTP Methods
---

For a REST API, your service will want to support different verbs/methods.
Http4s has a list of all the [methods] you're familiar with, and a few more.

```tut:book
import fs2.Task
import io.circe.generic._
import io.circe.syntax._
import org.http4s._, org.http4s.dsl._
import org.http4s.circe._

@JsonCodec case class TweetWithId(id: Int, message: String)
@JsonCodec case class Tweet(message: String)

def getTweet(tweetId: Int): Task[Option[TweetWithId]] = ???
def addTweet(tweet: Tweet): Task[TweetWithId] = ???
def updateTweet(id: Int, tweet: Tweet): Task[Option[TweetWithId]] = ???
def deleteTweet(id: Int): Task[Unit] = ???

implicit val tweetWithIdEncoder = jsonEncoderOf[TweetWithId]
implicit val tweetDecoder = jsonOf[Tweet]

val tweetService = HttpService {
  case GET -> Root / "tweets" / IntVar(tweetId) =>
    getTweet(tweetId)
      .flatMap(_.fold(NotFound())(Ok(_)))
  case req @ POST -> Root / "tweets" =>
    req.as[Tweet].flatMap(addTweet).flatMap(Ok(_))
  case req @ PUT -> Root / "tweets" / IntVar(tweetId) =>
    req.as[Tweet]
      .flatMap(updateTweet(tweetId, _))
      .flatMap(_.fold(NotFound())(Ok(_)))
  case req @ HEAD -> Root / "tweets" / IntVar(tweetId) =>
    getTweet(tweetId)
      .flatMap(_.fold(NotFound())(_ => Ok()))
  case req @ DELETE -> Root / "tweets" / IntVar(tweetId) =>
    deleteTweet(tweetId)
      .flatMap(Ok())
}
```

There's also [`DefaultHead`] which replicates the functionality of the native
implementation of the `HEAD` route.

[methods]: ../api/org/http4s/Method$.html
[`DefaultHead`]: ../api/org/http4s/server/middleware/DefaultHead$.html