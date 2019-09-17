# InfluxDB--操作，实现，踩坑

最近在处理公司埋点数据时发现了时序型数据库-influxDB
刚开始是使用Redis在做数据缓存，大部分数据都是需要与时间挂钩，使用Redis很不方便,则选择了influxDB。
偷懒：[介绍和使用](https://segmentfault.com/a/1190000015721780)


## golang操作influxDB
```
func QueryInfluxDB(conn client.Client, cmd string) (res []client.Result, err error) {
	q := client.Query{
		Command:  cmd,
		Database: Database,
	}
	if response, err := conn.Query(q); err == nil {
		if response.Error() != nil {
			return res, response.Error()
		}
		res = response.Results
	} else {
		return res, err

	}
	return res, nil
}

func Save2InfluxDB(typ, channel, name string, dataList []type) {
	conn := ConnInflux()
	defer conn.Close()
	tags := map[string]string{}

	bp, err := client.NewBatchPoints(client.BatchPointsConfig{
		Database:  GlobalInfluxDBConfig.Database,
		Precision: "s",
	})
	if err != nil {
		log.Println(err)
	}

	for _, data := range dataList {
		fields := map[string]interface{}{}

		p, err := client.NewPoint(name, tags, fields, time)
		if err != nil {
			log.Println(err)
			continue
		}

		bp.AddPoint(p)
	}

	if err := conn.Write(bp); err != nil {
		log.Println(err)
	}
}
```

## 踩坑
### partial write: points beyond retention policy dropped=XXX
```
写入数据的时间超出保存策略的时间
```