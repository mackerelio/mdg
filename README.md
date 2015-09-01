# mdg (mackerel dashboard generator)

### コマンド

```
$ mdg <orgName> <serviceName> <rollName> [ -f <config file(yaml)> ]
```

### 出力

markdown を標準出力にだす

### Usage

```
$ mdg <現状手動でorg名打ってもらう> `peco かなにかで service 絞り込み` `pecoかなにかで rollを絞り込み` -f ./config.yaml
```

### config.yaml

```yaml
- periods: # 省略される場合のデフォルトは 
  - 1 day
  - 1 week
  - 1 month
- metrics: # 省略される場合は全部表示
  - loadavg
  - cpu
  - ...
```

|key    |description                       |default             |
|-------|----------------------------------|--------------------|
|periods|グラフの表示期間のリスト          |x day y week z month|
|metrics|生成するグラフのメトリクスのリスト|全部 http://help-ja.mackerel.io/entry/spec/metrics |
