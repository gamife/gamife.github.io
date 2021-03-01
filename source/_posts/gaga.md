---
title: CAS
date:
categories: 
tags: 
---
## go 技巧

1. 使某个资源只会被关闭一次, 妙

   ```go
   // net/http/server.go:3360
   
   type onceCloseListener struct {
   	net.Listener					
   	once     sync.Once
   	closeErr error
   }
   
   // 使得这个 net.Listener只会被 close一次
   func (oc *onceCloseListener) Close() error {
   	oc.once.Do(oc.close)
   	return oc.closeErr
   }
   
   func (oc *onceCloseListener) close() { oc.closeErr = oc.Listener.Close() }
   
   ```


2. pool 加 reader转bufReader

   ```go
   // net/http/server.go:1794
   c.bufr = newBufioReader(c.r) 这一行进去
   
   bufio.NewReader(r io.Reader)  //io.Reader转有缓存的
   
   ```

// checkConnErrorWriter 这个包装可以看下, 如果Write有错误,将错误放进了 c.werr
   // 同时 c.cancelCtx()也被执行了
   c.bufw = newBufioWriterSize(checkConnErrorWriter{c}, 4<<10)

   ```
   
   

   ```

3. 判断是否超过最大值

   ```go
   
   ```

4. 链表, 以及取链表的一个元素

   ```go
   // runtime.notifyListWait()
   	if l.tail == nil {
   		l.head = s
   	} else {
   		l.tail.next = s
   	}
   	l.tail = s
   
   // runtime.notifyListNotifyOne()
   for p, s := (*sudog)(nil), l.head; s != nil; p, s = s, s.next {
   		if s.ticket == t {
   			n := s.next
   			if p != nil { //也就是把 s 取出来
   				p.next = n
   			} else {
   				l.head = n 
   			}
   			if n == nil {
   				l.tail = p
   			}
   			unlock(&l.lock)
   			s.next = nil
   			readyWithTime(s, 4)
   			return
   		}
   	}
   ```

   

5. panic

   ```go
   // x/sync/singleflight
   
   doCall() //处理panic
   ```


6. 有限制的并发

   ```go
   var wg sync.WaitGroup
   Ctxt.InParallel = true
   c := make(chan *Node, nBackendWorkers)
   for i := 0; i < nBackendWorkers; i++ {
       wg.Add(1)
       go func(worker int) {
           for fn := range c {
               compileSSA(fn, worker)
           }
           wg.Done()
       }(i)
   }
   for _, fn := range compilequeue {
       c <- fn
   }
   close(c)
   compilequeue = nil
   wg.Wait()
   ```

7. 基准测试

   ```go
   go test -gcflags=-N -benchmem -test.count=1 -test.cpu=1 -test.benchtime=1s -bench .
   
   https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/#425-%E5%8A%A8%E6%80%81%E6%B4%BE%E5%8F%91
   
   https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter09/09.2.html
   ```

8. 去除头或尾的空白

   ```go
   func isWhitespace(ch byte) bool { return ch == ' ' || ch == '\t' || ch == '\n' || ch == '\r' }
   
   func stripTrailingWhitespace(s string) string {
   	i := len(s)
   	for i > 0 && isWhitespace(s[i-1]) {
   		i--
   	}
   	return s[0:i]
   }
   
   ```

9. 反射改slice

   ```go
   slice := []string{"a","b","c"}
   v := reflect.Indirect(reflect.ValueOf(slice)).FieldByName("fieldName")	
   l := v.Len()
   reflect.Copy(v.Slice(i, l), v.Slice(i+1, l))
   v.Index(l - 1).Set(reflect.Zero(v.Type().Elem()))
   v.SetLen(l - 1)
   
   ```

如果是在一个遍历中使用反射改变切片长度, range的index还是原来的,所以要对改变长度的操作技术

   ```go
   
    var currentRemove bool
   			var preKey string
   			var nextRemove bool
   			var deleteCount int // 通过反射改变切片长度,range的index还是原来的
   			for i, k := range j.Decs {
   				currentRemove = nextRemove
   				nextRemove = false
   				hasFlag := strings.HasPrefix(k, flag)
   				if !hasFlag {
   					if currentRemove && k == "\n" {
   						nodeValue := reflect.ValueOf(node)
   						fieldValue := reflect.Indirect(nodeValue).FieldByName("Decs")
   						v := fieldValue.FieldByName(j.Name)
   						v = reflect.Indirect(v)
   						j.Decs = append(j.Decs[:i-deleteCount], j.Decs[i+1-deleteCount:]...) //去掉标志的注释, 但是是通过一个slice复制体,没有
   						v.SetLen(v.Len() - 1)
   						deleteCount++
   						WriteDst2File(node, filepath.Join(dir0, preKey+"_dst.txt")) // 不然会有个多余的 \n ,覆写一下
   					}
   				} else {
   					key := strings.TrimSuffix(k[len(flag):], suffix)
   					keys = append(keys, key)
   					//if key == "k2" {
   					//	fmt.Println(key)
   					//}
   					if commentDeleteFilter(key) {
   						nodeValue := reflect.ValueOf(node)
   						fieldValue := reflect.Indirect(nodeValue).FieldByName("Decs")
   						v := fieldValue.FieldByName(j.Name)
   						v = reflect.Indirect(v)
   						j.Decs = append(j.Decs[:i-deleteCount], j.Decs[i+1-deleteCount:]...) //去掉标志的注释, 但是是通过一个slice复制体,没有
   						v.SetLen(v.Len() - 1)
   						if strings.HasPrefix(flag, "/*") { //如果这一次有前缀,并且是/*开头的,要消除下一次的换行        
                
   ```

 