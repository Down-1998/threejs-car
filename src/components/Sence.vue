<template>
  <div id="three"></div>
  <Popover
    ref="popoverRef"
    :top="popoverTop"
    :left="popoverLeft"
    :data="popoverData"
  ></Popover>
</template>

<script lang="ts" setup name="Sence">
/* eslint-disable */
import { ref, onMounted, type Ref } from 'vue';
import Viewer, { type Animate } from '@/modules/Viewer';
import Floors from '@/modules/Floors';
import ModelLoader from '@/modules/ModelLoder';
import * as THREE from 'three';
import gsap from 'gsap';
import Event from '@/modules/Viewer/Events';
import BoxHelperWrap from '@/modules/BoxHelperWrap';
import { checkNameIncludes, findParent } from '@/utils';
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
import * as TWEEN from 'three/examples/jsm/libs/tween.module.js'

import Popover from './Popover/index.vue';

let viewer: Viewer;
let modelLoader: ModelLoader;
let boxHelperWrap: BoxHelperWrap;

const popoverRef: Ref = ref(null);
const popoverTop = ref(0);
const popoverLeft = ref(0);
const popoverData = ref<any>({});

let office: any = null;
let oldOffice: any = null;
let dataCenter: any = null;
let oldDataCenter: any = null;
let modelSelect = ['zuo0', 'zuo1', 'zuo2', 'zuo3', 'zuo4', 'zuo5'];
let modelSelectName = '';
let modelMoveName = '';
let isModelSelectName = false;
const doors:any = []
let bodyMaterial = new THREE.MeshPhysicalMaterial({
  color:'red',
  metalness:1,
  roughness:0.5,
  clearcoat:1.0,
  clearcoatRoughness:0.03
})
let glassMaterial = new THREE.MeshPhysicalMaterial({
  color:'#793e3e',
  metalness:0.25,
  roughness:0,
  transmission:1.0//透光性


})


onMounted(() => {
  init();
  initModel();

  viewer.scene.traverse((item: THREE.Object3D) => {
    console.log(item, '0000000000');
  });
});

const init = () => {
  viewer = new Viewer('three');
  viewer.addAxis();
  viewer.addStats();
  viewer.initRaycaster();
  initGUI();

  modelLoader = new ModelLoader(viewer);
  // const floors = new Floors(viewer);
  // floors.addGird();

  boxHelperWrap = new BoxHelperWrap(viewer);

  viewer.emitter.on(Event.dblclick.raycaster, (list: THREE.Intersection[]) => {
    onMouseClick(list);
  });

  viewer.emitter.on(Event.mousemove.raycaster, (list: THREE.Intersection[]) => {
    onMouseMove(list);
  });

};
const initGUI = () =>{
  let obj = {
    bodyColor:'#6e2121',
    glassColor:'#793e3e',
    caropen,
    carclose,
    carIn,
    carOut
  }
  let gui = new GUI();
  gui.addColor(obj,'bodyColor').name('车身颜色').onChange((value:string) =>{
    bodyMaterial.color.set(value)
  })
  gui.addColor(obj,'glassColor').name('车窗颜色').onChange((value:string) =>{
    glassMaterial.color.set(value)
  })
  gui.add(obj,'caropen').name('打开车门')
  gui.add(obj,'carclose').name('关闭车门')
  gui.add(obj,'carIn').name('车内部');
  gui.add(obj,'carOut').name('车外部');
}
const caropen = () =>{
 
  for(let i = 0; i < doors.length; i++){
    console.log(doors[i].rotation,'doors[i].rotation');
    setAnimationDoor({x:0},{x: Math.PI / 3},doors[i])
  }
}
const carclose = () =>{
  for(let i = 0; i < doors.length; i++){
    setAnimationDoor({x: Math.PI / 3},{x:0},doors[i])
  }
}
const carOut = () =>{
  setAnimationCamera({ cx: -0.27, cy: 0.83, cz: 0.6, ox: 0, oy: 0.5, oz: -3 }, { cx: 10, cy: 5, cz: 3, ox: 0, oy: 0.5, oz: 0 });
}
const carIn = () =>{
  setAnimationCamera({ cx: 4.25, cy: 1.4, cz: -4.5, ox: 0, oy: 0.5, oz: 0 }, { cx: -0.27, cy: 0.83, cz: 0.60, ox: 0, oy: 0.5, oz: -3 });
}
// 相机和控制器的移动
const setAnimationCamera = (start:any, end:any) =>{
  const tween = new TWEEN.Tween(start).to(end,3000).easing(TWEEN.Easing.Quadratic.Out);
  tween.onUpdate((that) =>{
   viewer.camera.position.set(that.cx,that.cy,that.cz);
   viewer.controls.target.set(that.ox,that.oy,that.oz);
  })
  tween.start();
}
// 车门的动画
const setAnimationDoor = (start:any,end:any,mesh:any) =>{
  // .easing(TWEEN.Easing.Quadratic.Out)
  const tween = new TWEEN.Tween(start).to(end,1000).easing(TWEEN.Easing.Quadratic.Out);
  console.log(tween,'tween');
  
  tween.onUpdate((that) =>{
    mesh.rotation.x = that.x;
  })
  tween.start();
}
const initModel = () => {
  // modelLoader.loadModelToScene('/models/zuo.glb', baseModel => {
  //   console.log(baseModel, '1111111');
    
  //   baseModel.setScalc(0.01);
  //   const model = baseModel.gltf.scene;
  //   office = baseModel;
  //   office.object.rotation.y = Math.PI;
  //   office.object.position.set(2, 0, 0);
  //   // model.position.set(80, 2, 90);
  //   office.object.children.forEach((item: any) => {
  //     item.name = item.name.replace('zuo', '');
  //     if (item.name === 'ding') {
  //       item.name = 6;
  //     }
  //     item.name--;
  //   });
  //   office.object.children.sort((a: { name: number; }, b: { name: number; }) => a.name - b.name).forEach((v: { name: string; }) => {
  //     v.name = 'zuo' + v.name;
  //   });

  //   model.name = '办公楼';
  //   baseModel.openCastShadow();
  //   oldOffice = model.clone();

  //   const list: THREE.Object3D<THREE.Event>[] = [];
  //   model.traverse(item => {
  //     list.push(item);
  //   });
  //   viewer.setRaycasterObjects(list);
  // });

  modelLoader.loadModelToScene('/models/plane.glb', baseModel => {
    console.log(baseModel,'baseModel plane');
    
    const model = baseModel.gltf.scene;
    model.scale.set(0.0001 * 3, 0.0001 * 3, 0.0001 * 3)
    model.position.set(0, 0, 0);
    model.name = 'plane';
    baseModel.openCastShadow();

    const texture = (baseModel.object.children[0] as any).material.map;
    console.log(texture, 'texture-------');
    const fnOnj = planeAnimate(texture);
    viewer.addAnimate(fnOnj);
  });
  

  // modelLoader.loadModelToScene('/models/datacenter.glb', baseModel => {
  //   console.log(baseModel, 'baseModel 机房');
  //   baseModel.setScalc(0.3);
  //   // baseModel.object.rotation.y = Math.PI / 2;
  //   const model = baseModel.gltf.scene;
  //   model.position.set(-1.5, 0, 0);
  //   model.name = '机房';
  //   baseModel.openCastShadow();

  //   dataCenter = baseModel;
  //   oldDataCenter = model.clone();

  //   const rackList: any[] = [];
  //   model.traverse(item => {
  //     if (checkIsRack(item)) {
  //       rackList.push(item);
  //     }
  //   });
  //   viewer.setRaycasterObjects(rackList);
  // });

  // modelLoader.loadModelToScene('/models/datacenter.glb', baseModel => {
  //   console.log(baseModel, 'baseModel 机房');
  //   baseModel.setScalc(0.3);
  //   const model = baseModel.gltf.scene;
  //   model.position.set(1.5, 0, 0);
  //   model.name = '机房';
  //   baseModel.openCastShadow();

  //   dataCenter = baseModel;
  //   oldDataCenter = model.clone();

  //   const rackList: any[] = [];
  //   model.traverse(item => {
  //     if (checkIsRack(item)) {
  //       rackList.push(item);
  //     }
  //   });
  //   viewer.setRaycasterObjects(rackList);
  // });
  // 徽章模型加载
  // modelLoader.loadModelToScene('/models/badge.glb',baseModel =>{
  //   console.log(baseModel,'baseModel');
    
  //   // baseModel.setScalc(0.5);
  //   const model = baseModel.gltf.scene;
  //   model.position.set(0,2,0);
  //   model.name = '徽章';
  //    baseModel.openCastShadow();
  // })

  // 徽章模型加载
  // modelLoader.loadModelToScene('/models/phone.glb',baseModel =>{
  //   console.log(baseModel,'baseModel iphone13');
    
  //   baseModel.setScalc(0.002);
  //   const model = baseModel.gltf.scene;
  //   model.position.set(3,0,3);
  //   model.name = 'iphone13';
  //    baseModel.openCastShadow();
  // })
// 动画模型加载
  let mixerDancer = null;
  modelLoader.loadModelToScene('/models/dancer.glb',baseModel =>{
    console.log(baseModel,'baseModel 跳舞小男孩');
    baseModel.startAnima();  
    baseModel.setScalc(0.5);
    const model = baseModel.gltf.scene; 
    model.position.set(0,0,3);
    model.name = '跳舞小男孩';
     baseModel.openCastShadow();
  })


  modelLoader.loadModelToScene('/models/911acr.glb',baseModel =>{
    console.log(baseModel,'保时捷911');
    baseModel.setScalc(0.7);
    const model = baseModel.gltf.scene;
    model.rotation.y = Math.PI
    model.traverse(((obj:any) =>{
      if(obj.name === 'Object_103' || obj.name == 'Object_64' || obj.name == 'Object_77'){
        console.log(obj,'911');
        obj.material = bodyMaterial
      }else if(obj.name === 'Object_90'){
        obj.material = glassMaterial
      }else if(obj.name === 'Empty001_16' || obj.name === 'Empty002_20'){
        doors.push(obj);
      }
      
    }))
    model.position.set(0,0,-0.5);
    model.name = '保时捷911';
    baseModel.openCastShadow();
  })
};

const planeAnimate = (texture: any): Animate => {
    console.log(texture, 'texture');
    texture.wrapS = THREE.RepeatWrapping;
    texture.wrapT = THREE.RepeatWrapping;
    const animateFn = {
      fun: () => {
        const count = texture.repeat.y;
        if (count <= 10) {
          texture.repeat.x += 0.01;
          texture.repeat.y += 0.02;
        } else {
          texture.repeat.x = 0;
          texture.repeat.y = 0;
        }
      },
      content: viewer,
    };
    return animateFn;
}

const onMouseClick = (intersects: THREE.Intersection[]) => {
  if (!intersects.length) return;
  const selectedObject = intersects[0].object;

  let selectedObjectName = '';
  const findClickModel = (object: any) => {
    console.log(object, 'object');
    if (object.type === 'Group') {
      selectedObjectName = object.name;
    }
    if (object.parent && object.type !== 'Scene') {
      findClickModel(object.parent);
    }
  };
  findClickModel(selectedObject);
  console.log(selectedObjectName);

  // if (!selectedObjectName || !selectedObjectName.includes('办公楼')) {
  //   // this.scene.remove(this.label);
  //   return;
  // }

  // const selectedModel = viewer.scene.getObjectByName(selectedObjectName);
  console.log(selectedObject, 'selectedObject');
  
  // 点击楼房
  if (selectedObject.name.includes('zuo')) {
    
    selectOffice(selectedObject.parent);
  }
  
  // 点击其他区域
  if (!selectedObject.name.includes('zuo')) {
    if (!isModelSelectName && oldOffice) {
      let oldmodel = oldOffice.getObjectByName(modelMoveName);
      office.object.getObjectByName(modelMoveName).traverse(function (child: { isMesh: any; material: any; name: any; }) {
        if (child.isMesh) {
          child.material = oldmodel.getObjectByName(child.name).material;
        }
      });
    }
  }
};

function checkIsRack (obj: any): boolean {
  return checkNameIncludes(obj, 'rack');
}

const onMouseMove = (intersects: THREE.Intersection[]) => {
  if (!intersects.length) {
    popoverRef.value.setShow(false);
    boxHelperWrap.setVisible(false);
    return;
  }
  const selectedObject = intersects[0].object || {};
  let selectedObjectName = '';
  const findClickModel = (object: any) => {
    if (object.name.includes('rack')) {
      selectedObjectName = object.name;
      return;
    }
    if (object.parent) {
      findClickModel(object.parent);
    }
  };
  
  findClickModel(selectedObject);

  console.log(selectedObjectName, '--selectedObjectName---');
  console.log(selectedObject, '------selectedObject---------');
  const rack = findParent(selectedObject, checkIsRack);
  console.log(rack, '-------rack---------');
  if (rack) {
    
    boxHelperWrap.attach(rack);
    updateRackInfo(rack.name);
  }

  // if (!selectedObjectName || !selectedObjectName.includes('办公楼')) {
  //   // 重置模型
  //   // viewer.scene.children[viewer.scene.children.findIndex(o => o.name === '办公楼')] = office.object = oldOffice.clone();
  //   return;
  // }


  modelSelect.forEach((item: any) => {
    if (item === selectedObject.parent?.name) {
      modelMoveName = item;
      if (modelSelectName === modelMoveName) return;
      office.object.getObjectByName(item).traverse(function (child: { isMesh: any; material: THREE.MeshPhongMaterial; }) {
        if (child.isMesh) {
          child.material = new THREE.MeshPhongMaterial({
            side: THREE.DoubleSide,
            transparent: true,
            depthTest: false,
            depthWrite: true, // 无法被选择，鼠标穿透
            color: '',
            opacity: 0.3,
          });
        }
      });
    } else {
      if (!isModelSelectName && oldOffice) {
        let oldmodel = oldOffice.getObjectByName(item);
        office.object.getObjectByName(item).traverse(function (child: { isMesh: any; material: any; name: any; }) {
          if (child.isMesh) {
            child.material = oldmodel.getObjectByName(child.name).material;
          }
        });
      }
    }
  });

};

const updateRackInfo = (name: string) => {
  if (name) {
    popoverRef.value.setShow(true, { name });
    const event = viewer.mouseEvent as MouseEvent;
    popoverTop.value = event.y + 10;
    popoverLeft.value = event.x + 10;
  } else {
    popoverRef.value.setShow(false);

  }
};

const selectOffice = (model: any) => {
  modelSelectName = model.name;
  let oldmodel = oldOffice.getObjectByName(modelSelectName);
  let modelSelectIndex = modelSelect.findIndex(v => v === modelSelectName);
  office.object.children.forEach((child: any, index: number) => {
    child.children.forEach((Mesh: any) => {
      if (child.name === modelSelectName) {
        child.children.forEach((Mesh: { material: any; name: any; }) => {
          Mesh.material = oldmodel.getObjectByName(Mesh.name).material;
        });
      } else {
        // Mesh.material = new THREE.MeshPhongMaterial({
        //   color: new THREE.Color('#123ca8'),
        //   transparent: true,
        //   opacity: 0.5,
        //   emissiveMap: Mesh.material.map,
        // });
      }
    });
    if (!model.userData.position && index > modelSelectIndex) {
      gsap.to(child.position, {
        y: !child.userData.position ? child.position.y + 60 : child.position.y,
        duration: 2,
        ease: "power1.inOut",
        onComplete: () => {
          child.userData.position = true;
        },
      });
    }
    if (model.userData.position && index <= modelSelectIndex) {
      if (child.userData.position) {
        gsap.to(child.position, {
          y: oldOffice.getObjectByName(child.name).position.y,
          duration: 2,
          ease: "power1.inOut",
          onComplete: () => {
            child.userData.position = false;
          },
        });
      }
    }
  });
};

</script>

<style scoped>
#three {
  height: 100%;
  width: 100%;
}
</style>
