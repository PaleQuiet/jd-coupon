# jd-coupon
抢京东优惠券(超级百亿补贴)，脚本使用时间2019/11
## 一、手动刷新脚本

1. 查看需要的优惠券标题，修改keywords关键词
2. 打开浏览器控制台，复制代码粘贴至控制台
3. 等待时间到达秒杀时间，在控制台刷新页面（复制的代码还会存在），回车运行

**code**
```
const keywords = ['鸡蛋', '爱媛38号果冻橙柑橘', '京觅 精品琯溪蜜柚']; // 想要的关键词

const part1 = document.querySelector('#NavPart1'); // 列表容器


function timeout(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms)
  })
}
async function asyncFun() {
  for (let i = 0; i < 18; i++) {
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
asyncFun()

```
## 二、自动刷新脚本，但是不知道是不是frame的原因，页面不滚动，有兴趣的可以改改试试
**code**
```
(() => {
    const keywords = ['鸡蛋', '爱媛38号果冻橙柑橘', '京觅 精品琯溪蜜柚', '一转三插座']; // 想要的关键词
    const moument = '2019-11-06 10:00'; // 抢购时间

    const ms = +new Date(moument); // 抢购时间毫秒值
    const pre = +new Date(ms - 1000 * 5); // 提前 5 秒钟准备
    const end = +new Date(ms + 1000 * 60 * 3); // 三分钟之后结束

    const part1 = document.querySelector('#NavPart1'); // 列表容器

    // 滚动百亿补贴列表，使其全部加载出来
    const scroll = () => {
        for (let i = 0; i < 18; i++) {
            setTimeout(() => {
              part1.scrollIntoView(false);
            }, 100 * i);
        }
    };

    // 获取服务器时间戳
    const getServerTime = async () => {
        return +new Date((await fetch(`?${ +new Date() }`, {
            method: 'head',
        }).then((res) => res.headers)).get("Date"));
    };

    const intervalId = setInterval(() => {
        console.log(`it is running...`);
        getServerTime().then(now => {
            console.log(`server time: ${ now }`);
            // 结束了
            if (now >= end) {
                console.info(`it's time to end.`);
                clearInterval(intervalId);
                return;
            }
            // 准备时间
            if (now >= pre) {
                console.log(`it's time to go...`);
                if (now >= ms) {
                    // 到点页面不会自动刷新，需要手动加载一下
                    let current = location.href;
                    let frame = '<frameset cols=\'*\'>\n<frame src=\'' + current + '\'/>';
                    frame += '</frameset>';
                    document.write(frame);
                    setTimeout(scroll, 600);
                    const items = part1.querySelectorAll('div.BillionSubsidy-content-item-coupon');
                    for (let item of items) {
                        const name = item.querySelector('div.BillionSubsidy-content-item-coupon-name');
                        for (let key of keywords) {
                            if (name.innerText.includes(key)) {
                                console.log(`key: [${ key }], matched: [${ name.innerText }]`);
                                const right = item.querySelector('div.BillionSubsidy-content-item-coupon-right-container');
                                const down = item.querySelector('div.BillionSubsidy-content-item-coupon-right-down');
                                if (right.innerText.includes('限量') && !right.innerText.includes('开抢')) {
                                    console.log(`wanna [${ key }], matched ${ name.innerText }`);
                                    if (down) {
                                        console.info(`--- ${ name.innerText } ---`);
                                        down.click();
                                        // down.remove();
                                        console.info(`*** u get it ***`);
                                    }
                                }
                            }
                        }
                    }
                }
            }
        });
    }, 800);
})();
```
