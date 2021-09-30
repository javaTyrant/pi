Java 7引入了，[ThreadLocalRandom](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/9b8c96f96a0f/src/share/classes/java/util/concurrent/ThreadLocalRandom.java#l64)以在竞争激烈的环境中提高随机数生成的吞吐量。

背后的原理ThreadLocalRandom很简单：Random每个线程都维护自己的版本，而不是共享一个全局实例Random。反过来，这减少了争用，从而提高了吞吐量。

