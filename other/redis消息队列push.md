# redis消息队列push/pop实现

## 连接redis
```
c, err := redis.Dial("tcp", "127.0.0.1:6379")
if err != nil {
    log.Println(err)
}
defer c.Close()
```

## push 
```
// push操作是使用redis中的LPUSH和RPUSH来实现
key := "test"
bs, err := json.Marshal(job)
if err != nil {
    log.Println(err)
    return
}
c.Do("lpush", key, bs)
```

## BRPOP/BLPOP
```
values, err := redis.Values(c.Do("BRPOP", key, 0))
if err != nil {
    log.Println(err)
}

var job Job

if len(values) > 2 {
    // 返回的是一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。
    reply := values[1]
    decoder := json.NewDecoder(bytes.NewReader(reply.([]byte)))
    err := decoder.Decode(&job)
    if err != nil {
        log.Println(err)
        return nil
    }
    return &job
} else {
    return nil
}
```