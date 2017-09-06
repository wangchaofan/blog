#断点上传实践
第一次发文，文笔很烂，技术很渣，大佬们不喜勿喷。55555

## 背景

一次去面试，给自己挖了一个坑，被问到有没有做到断点上传，知不知道其中的原理，瞬间进入懵逼状态。下来后去网上查了一些资料，写了一个简单的demo。

##原理
断点上传，顾名思义，一个文件可以分多次上传。核心点就是前端将文件分块传给后端，后端再进行拼接，最后完整文件上传。
> html5提供了FileApi, 可以用file.slice将文件分块，不支持html5的可以用flash实现

##代码
前端代码

```javascript
function upload(file) {
  var cache = 0
  var thunk = 1024
  var totalSize = file.size
	
  send()
	
  function send() {
    var form = new FormData()
    form.append('file', file.slice(cache, cache + thunk))
    form.append('fileName', file.name)
    ajax(form, function(response) {
      cache = cache + thunk
      document.getElementById('uploadProgress').innerText = (cache / totalSize > 1 ? 1 : cache / totalSize) * 100 + '%'
      if (cache < totalSize) {
        send()
      }
    })
  }
}
	
function ajax(data, cb) {
  var xhr = new XMLHttpRequest()
  xhr.open('post', '/upload')
  xhr.send(data)
  xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
      if (xhr.status === 200) {
        cb && cb(xhr.reponse)
      }
    }
  }
}
```

后端代码

``` javascript
const file = ctx.request.body.files.file
const fileName = ctx.request.body.fields.fileName
const reader = fs.createReadStream(file.path)
reader.on('data', function(chunk) {
  // 暂停读取
  reader.pause()
  fs.appendFile(path.join(__dirname, 'upload', fileName), chunk, function(err) {
    if (err) {
      ctx.body = 'error'
    }
    // 继续读取
    reader.resume()
  })
})

reader.end(function() {
	ctx.body = 'completed'
})
```

##总结
当然功能还存在很多缺陷，比如怎么实现多个块同时上传，手动暂停，保存已上传的状态等等，后面再慢慢完善。

###### 走刀口（Chris Wong）

