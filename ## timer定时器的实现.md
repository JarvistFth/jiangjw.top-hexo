## timer定时器的实现

### timerManager

负责管理timer，通过它可以添加timer，删除timer；

首先它要create一个timerfd，开启一个channel供EPOLL监听；每次我们设定的timerfd到时后，epoll会返回这个fd和channel。

通过timerChannel的ReadCallback()，这时我们获取所有过期的定时器(getExpired())，然后对每一个定时器绑定的回调函数进行调用(timer->run())；

timerManager只需创建一个timerfd就可以了，当我们添加timer的时候，我们将timer插入到set中，然后查看第一个过期的timer有没有变化，如果有变化，我们用最新插入的timer的过期时间来更新timerfd的值。

