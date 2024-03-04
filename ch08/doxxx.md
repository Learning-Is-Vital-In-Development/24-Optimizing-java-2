# Ch.8 GC 로깅, 모니터링, 튜닝, 툴

## GC 로깅 플래그

- GC 로그 순환 플래그

`-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=2M"`

```Bash
gc.log.0 — oldest
gc.log.1
gc.log.2
gc.log.3
gc.log.4 — latest
```
