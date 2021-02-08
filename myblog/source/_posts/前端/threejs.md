---
cover: /img/javascript.jpg
---


前端发展的某个方向，3d可视化开发。挺有兴趣。但是没空玩！随便记录下遇到的坑吧！

第一个坑：
直接看代码。new THREE.TextureLoader().load 加载纹理图片是一个异步方法。必须在成功回调中重新加载渲染器。否则，会出现已渲染完成才加载纹理图，导致的图片变黑色的问题。
```
  // 魔方体
    addCube: function () {
      //创建一个接受光照并带有纹理映射的立方体，并添加到场景中

      //加载贴图图片;
      let texture = new THREE.TextureLoader().load(imgurl, (texture) => {
        console.log("texture", texture);
        this.renderer.render(this.scene, this.camera);
      });

      let material = new THREE.MeshPhongMaterial({ map: texture });
      //创建一个立方体的几何体
      let geometry = new THREE.CubeGeometry(1, 1, 1);
      //将集合体和材质放到一个网格中
      this.mesh = new THREE.Mesh(geometry, material);
      //将立方体网格添加到场景中
      this.scene.add(this.mesh);
    },
```