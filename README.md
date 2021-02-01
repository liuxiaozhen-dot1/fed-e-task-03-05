

响应式系统升级

vue.js2.x中响应式系统的核心defineProperty

vue.js 3.x 中使用Proxy对象重写响应式系统

可以监听动态新增的属性  可以监听删除的属性  可以监听数组的索引和length属性

编译优化

vue.js 2.x中通过标记静态根节点，优化diff的过程

vue.js3.0中标记和提升所有的静态根节点，diff只要对比动态节点的内容





```js 
//setup是composition API 的入口

//createApp 创建vue对象

const app = createApp({

setup(){

//第一个参数是props

//第二个参数是context 分别有三个成员分别是attrs  emit slots

}

//setup会返回一个对象 可用于模板computed  methods以及生命周期的钩子函数中

//setup执行时间  props被解析完毕  setup内部不能获取组件实例  实例没有被创建

})



app.mount('#app')  // 挂载

reactive把对象转换成响应式对象 

toRefs这个对象属性也会被转换成响应式对象  参数是代理对象

ref把基本类型的数据转换成响应式对象

computed 创建一个响应式的数据，这个响应式 的数据依赖于其他响应式的数据，当依赖 的数据发生变化的时候 会重新计算这个数据传入的函数

watch 监听响应式数据的变化获取监听数据的新值和旧值

watch 有三个参数 

第一个参数 要监听的数据

第二个参数 监听到数据变化后执行的函数，这个函数有两个参数分别是新值和旧值

第三个参数 选项对象 deep和immediate

watch的返回值  取消监听的函数

watchEffect  是watch函数的简化版本 也是用来监听数据的变化

接受一个函数作为参数，监听函数内响应式数据的变化

vue3.0响应式回顾



Proxy对象实现属性监听

多层属性嵌套，在访问属性过程中处理下一级属性

默认监听动态添加属性

默认监听属性的删除操作

默认监听数组索引和length属性

可以作为单独的模块使用

核心函数 reactive/ref/toRefs/computed

effect  track  trigger 

reactive函数  接收一个参数，判断这个参数是否是对象

创建拦截器对象handler，设置get/set/deleteProperty

返回proxy对象

    //用来判断是否是对象
    const isObject = val=>{
        val!==null&&typeof val === 'object'
    }
    //判断返回的数据中是否数据类型为对象
    const convert = target=>isObject(target)?reactive(target):target
    //判断对象本身是否有某个属性
    const hasOwnProperty = Object.prototype.hasOwnProperty
    //判断tartget中是否有key属性
    const hasOwn = (target,key)=>hasOwnProperty.call(target,key)
    export function reactive(target){
        //如果不是对象直接返回target
        if(!isObject(target)){
            return target
        }
        const handler = {
            get(target,key,receiver){
                //收集依赖
                track(target,key)
                const result = Reflect.get(target,key,receiver)
                return convert(result)
            },
            set(target,key,value,receiver){
                //获取旧值 判断与新值是否相同
                const oldValue = Reflect.get(target,key,receiver)
                const result = true
                if(oldValue!==value){
                    result=Reflect.set(target,key,value,receiver)
                    //触发更新
                    trigger(target,key)
                }
                //返回一个布尔类型
                return result
            },
            deleteProperty(target,key){
                const hadKey = hadOwn(target,key)
                //删除对象属性中的key
                const result = Reflect.deleteProperty(target,key)
               //如果有删除
                if(hadKey&&result){
                   trigger(target,key)
                }
                return result
            }
        }
        return new Proxy(target,handler)
    }

effect 接收一个函数作为参数

    effect(()=>{
        
    })

收集依赖 effect track

    let activeEffect = null
    export function effect(callback){
        activeEffect = callback // 将依赖存储起来
        callback() //返回函数访问响应式对象属性，去收集依赖
        activeEffect = null //有嵌套函数 是个递归的过程
    }

track 有两个参数 对象和对象中的属性、

    let targetMap = new WeakMap()
    export function track (target,key){
        if(!activeEffect) return //如果没有依赖return
        let depsMap = targetMap.get(target) //去找依赖对象
        //如果没找到
        if(!depsMap){
            //创建一个新的集合添加到targetMap中
            targetMap.set(target,(depsMap=new Map()))
        }
        //查找属性
        let dep = depsMap.get(key)
        if(!dep){
            //创建一个新的集合添加到depsMap中
            depsMap.set(key,(dep= new Set()))
        }
        dep.add(activeEffect)
            
    }
    //把track放在reactive函数中

    //trigger有两个参数 target和key
    export function trigger(target,key){
        //找对象
        const depsMap = targetMap.get(target)
        //如果没有找到
        if(!depsMap) return 
        //如果找到了再根据key去找集合
        const dep = depsMap.get(key)
        //判断dep集合是否有值，遍历每个key
        if(dep){
            dep.forEach(effect=>{
                effect()
            })
        }
    }
    //把trigger函数放在reactive中触发set和deleteProperty的更新

ref  

      export function ref(raw){
          //判断raw是否是ref创建的对象，如果是的话直接返回
          if(isObject(raw)&&raw._v_isRef){
              return
          }
          let value = convert(raw)
          const r ={
              _v_isRef:true,
              get value(){
                  track(r,'value')
                  return value
              },
              set value(newValue){
                  if(newValue!==value){
                      raw = newValue
                      value = convert(raw)
                      trigger(r,'value')
                  }
              }
          }
          return r
      }

reactive 和 ref 对比

ref可以把基本数据类型根据类型数据，转成响应式对象

ref返回的对象，重新赋值成对象也是响应式的

reactive返回的对象，重新复制丢失响应式

reactive返回的对象不可以解构

toRefs   响应式原理

    export function toRefs(proxy){
        //判断是否是reactive创建的对象，如果不是会警告
       const net = proxy instance Array?new Array(proxy.length):{}
       //遍历
        for(const key in proxy){
            net[key] = toProxyRef(proxy,key)
        }
        return net
    }
    function toProxyRef(proxy,key){
        const r = {
            _v_isRef:true,
            get value(){
                return proxy[key]
            },
            set value( newValue){
                proxy[key] = newValue
            }
        }
        return r
    }

computed

    export function computed(getter){
        const result = ref()
        effect(()=>(result.value = getter()))
        return result
    }


