### customEvent
给 quill 富文本编辑器 添加自定义事件

### 使用方法

下载js文件
```
git clone https://github.com/kscript/customEvent.git
```
  
使用示例:  
vue单文件组件
``` vue
<script>
import Quill from 'quill'
import CustomEvent from './CustomEvent';
Quill.register('modules/customEvent', CustomEvent);

export default {
  data () {
    return {
      options: {
        modules: {
          customEvent: {
            events: [
              {
                name: 'click',
                handler: event => {
                  // ...
                }
              }
            ]
          }
        }
      }
    }
  }
}
</script>
```
