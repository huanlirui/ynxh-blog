需求：
定时触发一个任务，此任务又是一个循环任务。

使用node-schedule
注意：一定要 给每个任务设置名字。百度是不可能百度到的。全是一堆相同的粘贴的！！！！建议到git的issus和stack去找写法。
双层定时器。第一层触发时间，第二次是循环间隔触发。
2个取消任务的方法。根据名字取消。根据配置来取消。每次开启定时时，先把之前在跑的同名任务全部干掉！

下面粘贴一下伪代码吧

// 生成新的定时任务
let interval = async (options) => {
  return new Promise(async (resolve) => {

    let date = new Date(Date.now() + 5000);

    console.log('任务已启动，将在5秒后运行');
    // 终止之前的定时任务
    await cancelTaskOptions(options);
    // cancelTaskName(`${options.name}_${options.id}_${date.getTime()}`);
    // cancelTaskName(`${options.name}_${options.id}`);
    // 按照固定格式，设定定时任务，这里使用每条数据的名称+唯一字段ID，作为任务名称存入定时任务列表中
    // 任务名称就是'名字_id'
    let toArray = options.to.split(',');

    let counter = 0;
    schedule.scheduleJob(`${options.name}_${options.id}`, date, async () => {
      console.log('定时时间到了，开始执行~~~~~~~~');
    

      schedule.scheduleJob(
        `${options.name}_${options.id}_polling`,
        `*/${options.interval} * * * * *`, //设置执行间隔
        async () => {
          console.log('我是定时任务中的循环任务，我开始执行了！');
          try {
            
          } catch (error) {
            console.log('定时器触发次数：' + counter);
            counter++;
          }
        }
      );
    });

    // setTimeout(() => {
    //     editMaintainTime(options)
    //     console.log('任务结束了');
    // }, 10000);
  });
};

async function cancelTaskName(taskName) {
  // 查看所有的定时任务
  for (let i in schedule.scheduledJobs) {
    console.error('任务删除前：' + i);
  }
  if (schedule.scheduledJobs[`${taskName}`]) {
    console.log('终止定时任务', `${taskName}`);
    await schedule.scheduledJobs[`${taskName}`].cancel();
  }
  console.log('根据任务名称删除成功');
  // 查看剩下的定时任务
  for (let i in schedule.scheduledJobs) {
    console.error('任务删除后：' + i);
  }
}
async function cancelTaskOptions(options) {
  // 查看所有的定时任务
  for (let i in schedule.scheduledJobs) {
    console.error('任务删除前：' + i);
  }
  // 终止之前的定时任务
  if (schedule.scheduledJobs[`${options.name}_${options.id}_polling`]) {
    console.log('终止定时循环任务:', `${options.name}_${options.id}_polling`);
    await schedule.scheduledJobs[`${options.name}_${options.id}_polling`].cancel();
  }
  if (schedule.scheduledJobs[`${options.name}_${options.id}`]) {
    console.log('终止定时任务', `${options.name}_${options.id}`);
    await schedule.scheduledJobs[`${options.name}_${options.id}`].cancel();
  }
  // 查看剩下的定时任务
  for (let i in schedule.scheduledJobs) {
    console.error('任务删除后：' + i);
  }
}
