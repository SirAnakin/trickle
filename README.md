trickle
=======

A simple library for composing asynchronous code. The main reason for it to exist is to make it
easier to create graphs of interconnected asynchronous calls, pushing the 'worrying about
concurrency' aspects into the framework rather than mixing it in with the business logic.

## Getting Started

Include the latest version of Trickle into your project:

```
    <dependency>
      <groupId>com.spotify</groupId>
      <artifactId>trickle</artifactId>
      <version>0.4</version>
    </dependency>
```

Name the parameters to your call graph:

```java
  public static final Name<String> KEYWORD = new Name("keyword", String.class);
  public static final Name<String> ARTIST = new Name("artist", String.class);
```

Define the code to be executed in the nodes of your graph:

```java

  Func1<String, List<Track>> findTracks = new Func1<String, List<Track>>() {
      @Override
      public ListenableFuture<List<Track>> run(String keyword) {
        return search.findTracks(keyword);
      }
  };
  Func1<String, Artist> findArtist = new Func1<String, Artist>() {
      @Override
      public ListenableFuture<Artist> run(String artistName) {
        return metadata.lookupArtist(artistName);
      }
  };
  Func2<Artist, List<Track>, MyOutput> combine = new Func2<Artist, List<Track>, MyOutput>() {
      @Override
      public ListenableFuture<MyOutput> run(Artist artist, List<Track> tracks) {
        return Futures.immediateFuture(new MyOutput(artist, tracks));
      }
  };
```

Wire up your call graph:

```java
  Graph<List<Track>> tracks = Trickle.call(findTracks).with(KEYWORD);
  Graph<Artist> artist = Trickle.call(findArtist).with(ARTIST);
  this.output = Trickle.call(combine).with(artist, tracks);
```

At some later stage, call the graph for some specific keyword and artist name:

```java
  public ListenableFuture<MyOutput> doTheThing(String keyword, String artistName) {
    return this.output.bind(KEYWORD, keyword).bind(ARTIST, artistName).run();
  }
```

See [Examples.java](src/examples/java/com/spotify/trickle/example/Examples.java) for more examples
and see [the wiki](/step/trickle/wiki) for more in-depth descriptions of the library.



