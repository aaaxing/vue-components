1.地点选择器插件，基于VUE，不依赖任何ui库和其他依赖。
2.如何使用：
将placePicker.vue拷如项目文件夹components下
在你需要用的父组件中引入import placePicker from 'placePicker'
components:{placePicker}
<placePicker titleName='起始地' titleContent='请输入起始地' v-model='localInfo'></placePicker>

3.placePicker中需要加载地址数据
import place from 'townplace'

4.注意，当前插件只用于四川地区，如果全国地区，后期会改进