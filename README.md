# fs2-cron
[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/fthomas/fs2-cron/Continuous%20Integration)](https://github.com/fthomas/fs2-cron/actions?query=workflow%3A%22Continuous+Integration%22)
[![codecov](https://codecov.io/gh/fthomas/fs2-cron/branch/master/graph/badge.svg)](https://codecov.io/gh/fthomas/fs2-cron)
[![Join the chat at https://gitter.im/fthomas/fs2-cron](https://badges.gitter.im/fthomas/fs2-cron.svg)](https://gitter.im/fthomas/fs2-cron?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Scaladex](https://index.scala-lang.org/fthomas/fs2-cron/latest.svg?color=blue)](https://index.scala-lang.org/fthomas/fs2-cron/fs2-cron-core)
[![Scaladoc](https://www.javadoc.io/badge/eu.timepit/fs2-cron-core_2.12.svg?color=blue&label=Scaladoc)](https://javadoc.io/doc/eu.timepit/fs2-cron-core_2.12)

**fs2-cron** is a microlibrary that provides [FS2][FS2] streams based
on [Cron4s][Cron4s] cron expressions.

## Examples

```scala
import cats.effect.IO
import cats.effect.unsafe.implicits.global
import cron4s.Cron
import eu.timepit.fs2cron.ScheduledStreams
import eu.timepit.fs2cron.cron4s.Cron4sScheduler
import fs2.Stream
import java.time.LocalTime
```
```scala
val streams = new ScheduledStreams(Cron4sScheduler.systemDefault[IO])
// streams: ScheduledStreams[IO[A], cron4s.expr.CronExpr] = eu.timepit.fs2cron.ScheduledStreams@2e16be8b

val evenSeconds = Cron.unsafeParse("*/2 * * ? * *")
// evenSeconds: cron4s.package.CronExpr = CronExpr(
//   seconds = */2,
//   minutes = *,
//   hours = *,
//   daysOfMonth = ?,
//   months = *,
//   daysOfWeek = *
// )

val printTime = Stream.eval(IO(println(LocalTime.now)))
// printTime: Stream[IO, Unit] = Stream(..)

val scheduled = streams.awakeEvery(evenSeconds) >> printTime
// scheduled: Stream[IO[x], Unit] = Stream(..)

scheduled.take(3).compile.drain.unsafeRunSync()
// 19:56:10.263
// 19:56:12.003
// 19:56:14.002
```
```scala
val everyFiveSeconds = Cron.unsafeParse("*/5 * * ? * *")
// everyFiveSeconds: cron4s.package.CronExpr = CronExpr(
//   seconds = */5,
//   minutes = *,
//   hours = *,
//   daysOfMonth = ?,
//   months = *,
//   daysOfWeek = *
// )

val scheduledTasks = streams.schedule(List(
  evenSeconds      -> Stream.eval(IO(println(LocalTime.now.toString + " task 1"))),
  everyFiveSeconds -> Stream.eval(IO(println(LocalTime.now.toString + " task 2")))
))
// scheduledTasks: Stream[IO[A], Unit] = Stream(..)

scheduledTasks.take(9).compile.drain.unsafeRunSync()
// 19:56:15.004 task 2
// 19:56:16.002 task 1
// 19:56:18.002 task 1
// 19:56:20.003 task 2
// 19:56:20.003 task 1
// 19:56:22.002 task 1
// 19:56:24.002 task 1
// 19:56:25.002 task 2
// 19:56:26.003 task 1
```

## Using fs2-cron

The latest version of the library is available for Scala 2.12 and 2.13.

If you're using sbt, add the following to your build:
```sbt
libraryDependencies ++= Seq(
  "eu.timepit" %% "fs2-cron-cron4s" % "0.6.0"
)
```

## License

**fs2-cron** is licensed under the Apache License, Version 2.0, available at
http://www.apache.org/licenses/LICENSE-2.0 and also in the
[LICENSE](https://github.com/fthomas/status-page/blob/master/LICENSE) file.

[Cron4s]: https://github.com/alonsodomin/cron4s
[FS2]: https://github.com/functional-streams-for-scala/fs2
