<template>
  <div>
    <!-- Toolbar for adding text, image, colors, etc. -->
    <el-row class="toolbar" :gutter="10">
      <el-col :span="1.5">
        <el-button icon="EditPen" @click="addText"> Text</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button icon="EditPen" @click="addIText"> IText</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-upload :show-file-list="false" accept="image/*" :before-upload="insertImage">
          <el-button icon="Picture"> Image</el-button>
        </el-upload>
      </el-col>
      <el-col :span="1.5" v-if="activeObject">
        <el-color-picker v-model="color" show-alpha @active-change="(c) => color = c" />
      </el-col>
      <el-col :span="1.5" v-if="activeObject">
        <el-color-picker v-model="itemBackgroundColor" show-alpha @active-change="(c) => itemBackgroundColor = c" />
      </el-col>
      <el-col :span="1.5" v-if="activeObject">
        <el-button icon="Delete" @click="eraseItem"> Delete</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-upload :show-file-list="false" accept="image/*" :before-upload="loadBackground">
          <el-button icon="PictureFilled"> Background Image</el-button>
        </el-upload>
      </el-col>
    </el-row>


    <!-- Canvas -->
    <div>
      <canvas id="atexCanvas"></canvas>
    </div>
  </div>
</template>

<script setup>
import { nextTick, onMounted, ref, watch } from 'vue';
import { fabric } from 'fabric';

const color = ref('#000000');
const itemBackgroundColor = ref('#ffffff');

const activeObject = ref(null);
let canvas;

var deleteIcon = "data:image/svg+xml,%3C%3Fxml version='1.0' encoding='utf-8'%3F%3E%3C!DOCTYPE svg PUBLIC '-//W3C//DTD SVG 1.1//EN' 'http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd'%3E%3Csvg version='1.1' id='Ebene_1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' x='0px' y='0px' width='595.275px' height='595.275px' viewBox='200 215 230 470' xml:space='preserve'%3E%3Ccircle style='fill:%23F44336;' cx='299.76' cy='439.067' r='218.516'/%3E%3Cg%3E%3Crect x='267.162' y='307.978' transform='matrix(0.7071 -0.7071 0.7071 0.7071 -222.6202 340.6915)' style='fill:white;' width='65.545' height='262.18'/%3E%3Crect x='266.988' y='308.153' transform='matrix(0.7071 0.7071 -0.7071 0.7071 398.3889 -83.3116)' style='fill:white;' width='65.544' height='262.179'/%3E%3C/g%3E%3C/svg%3E";
var cloneIcon = "data:image/svg+xml,%3C%3Fxml version='1.0' encoding='iso-8859-1'%3F%3E%3Csvg version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' viewBox='0 0 55.699 55.699' width='100px' height='100px' xml:space='preserve'%3E%3Cpath style='fill:%23010002;' d='M51.51,18.001c-0.006-0.085-0.022-0.167-0.05-0.248c-0.012-0.034-0.02-0.067-0.035-0.1 c-0.049-0.106-0.109-0.206-0.194-0.291v-0.001l0,0c0,0-0.001-0.001-0.001-0.002L34.161,0.293c-0.086-0.087-0.188-0.148-0.295-0.197 c-0.027-0.013-0.057-0.02-0.086-0.03c-0.086-0.029-0.174-0.048-0.265-0.053C33.494,0.011,33.475,0,33.453,0H22.177 c-3.678,0-6.669,2.992-6.669,6.67v1.674h-4.663c-3.678,0-6.67,2.992-6.67,6.67V49.03c0,3.678,2.992,6.669,6.67,6.669h22.677 c3.677,0,6.669-2.991,6.669-6.669v-1.675h4.664c3.678,0,6.669-2.991,6.669-6.669V18.069C51.524,18.045,51.512,18.025,51.51,18.001z M34.454,3.414l13.655,13.655h-8.985c-2.575,0-4.67-2.095-4.67-4.67V3.414z M38.191,49.029c0,2.574-2.095,4.669-4.669,4.669H10.845 c-2.575,0-4.67-2.095-4.67-4.669V15.014c0-2.575,2.095-4.67,4.67-4.67h5.663h4.614v10.399c0,3.678,2.991,6.669,6.668,6.669h10.4 v18.942L38.191,49.029L38.191,49.029z M36.777,25.412h-8.986c-2.574,0-4.668-2.094-4.668-4.669v-8.985L36.777,25.412z M44.855,45.355h-4.664V26.412c0-0.023-0.012-0.044-0.014-0.067c-0.006-0.085-0.021-0.167-0.049-0.249 c-0.012-0.033-0.021-0.066-0.036-0.1c-0.048-0.105-0.109-0.205-0.194-0.29l0,0l0,0c0-0.001-0.001-0.002-0.001-0.002L22.829,8.637 c-0.087-0.086-0.188-0.147-0.295-0.196c-0.029-0.013-0.058-0.021-0.088-0.031c-0.086-0.03-0.172-0.048-0.263-0.053 c-0.021-0.002-0.04-0.013-0.062-0.013h-4.614V6.67c0-2.575,2.095-4.67,4.669-4.67h10.277v10.4c0,3.678,2.992,6.67,6.67,6.67h10.399 v21.616C49.524,43.26,47.429,45.355,44.855,45.355z'/%3E%3C/svg%3E%0A"

// Initialize Fabric.js canvas on component mount
onMounted(async () => {
  canvas = new fabric.Canvas('atexCanvas', {
    width: 800,
    height: 600,
  });

  await nextTick();

  canvas.on('selection:created', () => {
    let selectedObject = canvas.getActiveObject();
    activeObject.value = selectedObject;
    // activeObject.value = canvas.getActiveObject();
    if (activeObject.value) {
      selectedObject.set({
        borderColor: 'red',
        cornerColor: 'green',
        cornerSize: 10,
        transparentCorners: false,
      });
      canvas.renderAll();
    }
  });

  canvas.on('selection:updated', () => {
    let selectedObject = canvas.getActiveObject();
    activeObject.value = selectedObject;

    if (activeObject.value) {
      selectedObject.set({
        borderColor: 'red',
        cornerColor: 'green',
        cornerSize: 10,
        transparentCorners: false,
      });
      canvas.renderAll();
    }
  });

  canvas.on('selection:cleared', () => {
    activeObject.value = null;
  });


  //Keydown Handlers
  window.addEventListener('keydown', handleKeyDown);
});

/**
 * Tranformation Controlers
 */
const loadControlls = () => {
  var deleteImg = document.createElement('img');
  deleteImg.src = deleteIcon;

  var cloneImg = document.createElement('img');
  cloneImg.src = cloneIcon;

  const renderIcon = (icon) => {
    return function renderIcon(ctx, left, top, styleOverride, fabricObject) {
      console.log(styleOverride);
      var size = this.cornerSize;
      ctx.save();
      ctx.translate(left, top);
      ctx.rotate(fabric.util.degreesToRadians(fabricObject.angle));
      ctx.drawImage(icon, -size / 2, -size / 2, size, size);
      ctx.restore();
    }
  }

  const deleteObject = (eventData, transform) => {
    var target = transform.target;
    var canvas = target.canvas;
    canvas.remove(target);
    canvas.requestRenderAll();
  }

  const cloneObject = (eventData, transform) => {
    var target = transform.target;
    var canvas = target.canvas;
    target.clone((cloned) => {
      cloned.left += 10;
      cloned.top += 10;
      canvas?.add(cloned);
    });

    return true;
  }

  fabric.Object.prototype.controls.deleteControl = new fabric.Control({
    x: -0.60,
    y: 0.5,
    offsetY: 5,
    offsetX: 5,
    cursorStyle: 'pointer',
    mouseUpHandler: deleteObject,
    render: renderIcon(deleteImg),
    cornerSize: 16
  });

  fabric.Object.prototype.controls.clone = new fabric.Control({
    x: 0.55,
    y: 0.5,
    offsetY: 5,
    offsetX: 5,
    cursorStyle: 'pointer',
    mouseUpHandler: cloneObject,
    render: renderIcon(cloneImg),
    cornerSize: 16
  });
}

/**
 * Add Text on the canvas
 */
const addText = () => {
  console.log(activeObject.value);

  const canvasCenter = getCenter();
  const text = new fabric.Textbox('Enter Text', {
    fill: color.value,
    backgroundColor: itemBackgroundColor.value,
    fontFamily: "Helvetica",
    selectable: true,
  });

  text.set({
    left: canvasCenter.x - ((text.width || 1) / 2),
    top: canvasCenter.y - ((text.height || 1) / 2),
  });

  canvas.add(text);
};

/**
 * add IText on the canvas
 */
const addIText = () => {
  console.log(activeObject.value);

  var canvasCenter = getCenter();
  const text = new fabric.IText('Enter Text', {
    fill: color.value,
    fontFamily: "Helvetica",
    selectable: true,
  });

  text.set({
    left: canvasCenter.x - ((text.width || 1) / 2),
    top: canvasCenter.y - ((text.height || 1) / 2),
  });
  canvas.add(text);
};

/**
 * Insert Image on the canvas
 * @param file image/*
 * @return false
 */
const insertImage = (file) => {
  const reader = new FileReader();

  reader.onload = (event) => {
    const imgUrl = event.target.result;

    fabric.Image.fromURL(imgUrl, (img) => {
      var canvasCenter = getCenter();

      img.set({ left: canvasCenter.x - (img.width / 2), top: canvasCenter.y - (img.height / 2) });
      canvas.add(img);
      console.log("Image successfully loaded and added to the canvas.");
    });
  };

  reader.onerror = (error) => {
    console.error("FileReader error:", error);
  };

  reader.readAsDataURL(file.raw || file);

  return false;
}


/**
 * Set Text Color
 */
const setActiveColor = () => {
  console.log(activeObject.value);
  if (activeObject.value) {
    activeObject.value.set({ fill: color.value });
    canvas.renderAll();
  }
};

/**
 * Set background color of the selected object
 */
const setBackgroundColor = () => {
  if (activeObject.value) {
    activeObject.value.set({ backgroundColor: itemBackgroundColor.value });
    canvas.renderAll();
  }
};

/**
 * Remove the selected object when 'Delete' or 'Backspace' key is pressed
 */
const eraseItem = () => {
  const obj = activeObject.value;  // Get the raw value of activeObject

  if (obj) {
    canvas.remove(canvas.getActiveObject());  // Directly remove the object
    canvas.renderAll();   // Rerender the canvas after removal

    console.log("Object removed successfully");
  } else {
    console.log("No active object found");
  }
};

/**
 * Calculate Canvas Center
 */
const getCenter = () => {
  var canvasCenterX = canvas.getWidth() / 2;
  var canvasCenterY = canvas.getHeight() / 2;

  return { x: canvasCenterX, y: canvasCenterY };
}

/**
 * Load an image as background and set it as clip path
 */
const loadBackground = (file) => {
  const reader = new FileReader();

  reader.onload = (event) => {
    const imgUrl = event.target.result;
    fabric.Image.fromURL(imgUrl, (img) => {
      // Calculate the center position
      var canvasCenter = getCenter();
      // Set img attributes
      img.set({
        left: canvasCenter.x - (img.width / 2),
        top: canvasCenter.y - (img.height / 2),
        selectable: false,
      });

      // Send Img to Back
      img.sendToBack();
      canvas.add(img);
      canvas.clipPath = img;
      canvas.requestRenderAll();
    });
  };

  reader.onerror = (error) => {
    console.error("FileReader error:", error);
  };

  reader.readAsDataURL(file.raw || file);

  return false;
}

//   // new fabric.loadSVGFromURL(`./shirts.svg`, function (objects, options) {
//   //   var svgData = fabric.util.groupSVGElements(objects, options);

//   //   // Calculate the center position
//   //   var canvasCenterX = canvas.getWidth() / 2;
//   //   var canvasCenterY = canvas.getHeight() / 2;

//   //   // Calculate the position to center the SVG
//   //   svgData.top = canvasCenterY - (svgData.height / 2);
//   //   svgData.left = canvasCenterX - (svgData.width / 2);

//   //   svgData.stroke = "red";
//   //   svgData.strokeWidth = 3;
//   //   svgData.selectable = false;
//   //   canvas.add(svgData);
//   //   canvas.clipPath = svgData;
//   // });

//   // canvas.add(shirtImage);
//   console.log(shirtImage);
//   console.log("Button clicked:after");

// }

loadControlls();

//keydown definition
const handleKeyDown = (e) => {
  if (e.key === 'Delete') {
    eraseItem();
  }
};

watch(
  () => color.value,
  () => setActiveColor()
);

watch(
  () => itemBackgroundColor.value,
  () => setBackgroundColor()
);

watch(
  () => activeObject.value,
  () => {

  }
);

</script>

<style scoped>
.toolbar {
  display: flex;
  flex-direction: row;
  margin-bottom: 10px;
}

canvas {
  border: 1px solid #ccc;
}

.inline {
  display: inline-block;
  margin: 0 5px 0 5px;
}
</style>
