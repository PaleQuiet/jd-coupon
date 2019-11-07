# jd-coupon
抢京东优惠券(超级百亿补贴)，脚本使用时间2019/11
## 需要油猴脚本配合使用

**code**
```
(function () {
  'use strict';
  const keywords = ['鸡蛋', '德国进口酸奶 德亚', '爱媛38号果冻橙柑橘', '云南玉溪高山冰糖橙', '陕西徐香绿心猕猴桃', '京觅 精品琯溪蜜柚', '金典有机纯牛奶250ml']; // 想要的关键词
  const loadNum = 6; // 第一次加载页面加载出来的优惠商品的数量
  const moument = '2019-11-06 10:00'; // 抢购时间

  const ms = +new Date(moument); // 抢购时间毫秒值
  const end = +new Date(ms + 1000 * 60 * 3); // 三分钟之后结束

  let isreload = false;

  // 页面监控，要等到开始加载优惠商品的时候再调用函数
  let listeners = [];
  const MutationObserver = window.MutationObserver || window.WebKitMutationObserver;
  let observer;

  // 获取服务器时间戳
  const getServerTime = async () => {
    return +new Date((await fetch(`?${ +new Date() }`, {
      method: 'head',
    }).then((res) => res.headers)).get("Date"));
  };

  const intervalId = setInterval(() => {
    getServerTime().then(now => {
      // 结束了
      if (now >= end) {
        console.info(`it's time to end.`);
        clearInterval(intervalId);
        return;
      }
      // 开始
      if (now >= ms) {
        if (now - ms < 610) {
          location.reload()
        }
        if (!isreload) {
          isreload = true;
          ready('#NavPart1 .BillionSubsidy-content-item-coupon', asyncFun);
        }
      }
    });
  }, 600);
  // 延时调用
  function timeout(ms) {
    return new Promise(resolve => {
      setTimeout(resolve, ms)
    })
  }
  async function asyncFun() {
    const part1 = document.querySelector('#NavPart1');
    // scrollIntoView 执行18次之后 再进行点击操作
    for (let i = 0; i < 18; i++) {
      // 滚动出所有商品
      part1.scrollIntoView(false);
      await timeout(10);
    }

    const items = part1.querySelectorAll('div.BillionSubsidy-content-item-coupon');
    for (let item of items) {
      const name = item.querySelector('div.BillionSubsidy-content-item-coupon-name');
      for (let key of keywords) {
        if (name.innerText.includes(key)) {
          const right = item.querySelector('div.BillionSubsidy-content-item-coupon-right-container');
          const down = item.querySelector('div.BillionSubsidy-content-item-coupon-right-down');
          if (right.innerText.includes('限量') && !right.innerText.includes('开抢')) {
            if (down) {
              down.click();
              console.info(`--- ${ name.innerText } ---`);
              // down.remove();
              console.info(`*** u get it ***`);
            }
          }
        }
      }
    }
  }

  function ready(selector, fn) {
    // 储存选择器和回调函数
    listeners.push({
      selector: selector,
      fn: fn
    });
    if (!observer) {
      // 监听document变化
      observer = new MutationObserver(check);
      observer.observe(document.documentElement, {
        childList: true,
        subtree: true
      });
    }
    // 检查该节点是否已经在DOM中
    check();
  }

  function check() {
    // 检查是否匹配已储存的节点
    for (var i = 0; i < listeners.length; i++) {
      var listener = listeners[i];
      // 检查指定节点是否有匹配
      var elements = document.querySelectorAll(listener.selector);

      if (elements.length >= loadNum) {
        listener.fn.call();
      }
    }
  }
})();
```
