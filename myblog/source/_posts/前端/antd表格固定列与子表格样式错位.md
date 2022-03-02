

在展开子表的回调函数（expand）里面添加重新计算的代码

tableclassname 为表格类名。css隔离
```
 this.$nextTick(() => {
    let element = document.querySelectorAll(
      ".tableclassname .ant-table-expanded-row"
      );
    let len = element.length / 2;
    for (let i = 0; i < len; i++) {
      element[i + len].style.height = element[i].offsetHeight + "px";
    }
  });

  ```