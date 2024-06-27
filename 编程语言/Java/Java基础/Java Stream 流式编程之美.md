# Java Stream 流式优化

## 遗留代码：找出长度大于 1 分钟的曲目

```java
            public Set<String> findLongTracks(List<Album> albums) {
                Set<String> trackNames = new HashSet<>();
                for(Album album : albums) {
                    for (Track track : album.getTrackList()) {
                        if (track.getLength() > 60) {
                            String name = track.getName();
                            trackNames.add(name);
                        }
                    }
                }
                return trackNames;
            }
```

<!-- more -->

### 重构的第一步：找出长度大于 1 分钟的曲目

```java
        public Set<String> findLongTracks(List<Album> albums) {
            Set<String> trackNames = new HashSet<>();
            albums.stream()
                .forEach(album -> {
                    album.getTracks()
                        .forEach(track -> {
                            if (track.getLength() > 60) {
                                String name = track.getName();
                                trackNames.add(name);
                            }
                        });
                });
            return trackNames;
        }
```

### 重构的第二步：找出长度大于 1 分钟的曲目

```java
        public Set<String> findLongTracks(List<Album> albums) {
            Set<String> trackNames = new HashSet<>();
            albums.stream()
                .forEach(album -> {
                    album.getTracks()
                        .filter(track -> track.getLength() > 60)
                        .map(track -> track.getName())
                        .forEach(name -> trackNames.add(name));
                });
            return trackNames;
        }
```

### 重构的第三步：找出长度大于 1 分钟的曲目

```java
        public Set<String> findLongTracks(List<Album> albums) {
            Set<String> trackNames = new HashSet<>();
            albums.stream()
                .flatMap(album -> album.getTracks())
                .filter(track -> track.getLength() > 60)
                .map(track -> track.getName())
                .forEach(name -> trackNames.add(name));
            return trackNames;
        }
```

### 重构的第四步：找出长度大于 1 分钟的曲目

```java
        public Set<String> findLongTracks(List<Album> albums) {
            return albums.stream()
                .flatMap(album -> album.getTracks())
                .filter(track -> track.getLength() > 60)
                .map(track -> track.getName())
                .collect(toSet());
        }
```

## Java Stream WET  流式表达式重构代码

不要重复你劳动（Don’t Repeat Yourself，DRY）是一个众所周知的模式，它的反面是同样

的东西写两遍（Write Everything Twice，WET）。

### 基础代码

增加一个简单的 Order 类来

计算用户购买专辑的一些有用属性，如计算音乐家人数、曲目和专辑时长等。如果使用命

令式 Java，编写出的代码如下

```java
  public long countRunningTime() {
    long count = 0;
    for (Album album : albums) {
      for (Track track : album.getTrackList()) {
        count += track.getLength();
      }
    }
    return count;
  }

  public long countMusicians() {
    long count = 0;
    for (Album album : albums) {
      count += album.getMusicianList().size();
    }
    return count;
  }

  public long countTracks() {
    long count = 0;
    for (Album album : albums) {
      count += album.getTrackList().size();
    }
    return count;
  }
```

### 使用流重构命令式的 Order 类

```java
  public long countRunningTime() {
    return albums.stream()
        .mapToLong(album -> album.getTracks()
            .mapToLong(track -> track.getLength())
            .sum())
        .sum();
  }

  public long countMusicians() {
    return albums.stream()
        .mapToLong(album -> album.getMusicians().count())
        .sum();
  }

  public long countTracks() {
    return albums.stream()
        .mapToLong(album -> album.getTracks().count())
        .sum();
  }
```

### 使用领域方法重构 Order 类

```java
  public long countFeature(ToLongFunction<Album> function) {
    return albums.stream()
        .mapToLong(function)
        .sum();
  }

  public long countTracks() {
    return countFeature(album -> album.getTracks().count());
  }

  public long countRunningTime() {
    return countFeature(album -> album.getTracks()
        .mapToLong(track -> track.getLength())
        .sum());
  }

  public long countMusicians() {
    return countFeature(album -> album.getMusicians().count());
  }
```
