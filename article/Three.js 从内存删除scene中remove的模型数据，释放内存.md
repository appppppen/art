当需要动态更新显示不同的模型，从 scene 中 remove 的模型还会在内存中，如果多次进行更新操作就会大量占用内存资源，甚至使浏览器崩溃，以下代码功能为从内存中清除模型数据。
``` typescript
/**
 * 清除模型，模型中有 group 和 scene,需要进行判断
 * @param scene
 * @returns
 */
clearScene() {
	// 从scene中删除模型并释放内存
	if(myObjects.length > 0){		
		for(var i = 0; i< myObjects.length; i++){
			var currObj = myObjects[i];
			// 判断类型
			if(currObj instanceof THREE.Scene){
				var children = currObj.children;
				for(var i = 0; i< children.length; i++){
					deleteGroup(children[i]);
				}	
			}else{				
				deleteGroup(currObj);
			}
			scene.remove(currObj);
		}
	}
}

// 删除group，释放内存
deleteGroup(group) {
    if (!group) return;
    // 删除掉所有的模型组内的mesh
    group.traverse( (item) =>{
        if (item instanceof THREE.Mesh) {
            item.geometry.dispose(); // 删除几何体
            item.material.dispose(); // 删除材质
        }
    });
}
```
