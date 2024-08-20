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
 * Add Text on the canvas
 */
const addText = () => {
  console.log(activeObject.value);

  const text = new fabric.Textbox('Enter Text', {
    left: 100,
    top: 100,
    fill: color.value,
    backgroundColor: itemBackgroundColor.value,
    fontFamily: "Helvetica",
    selectable: true,
  });
  canvas.add(text);
};

/**
 * add IText on the canvas
 */
const addIText = () => {
  console.log(activeObject.value);

  const text = new fabric.IText('Enter Text', {
    left: 100,
    top: 100,
    fill: color.value,
    // backgroundColor: itemBackgroundColor.value,
    fontFamily: "Helvetica",
    selectable: true,
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
      img.set({ left: 100, top: 100 });
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
 * Load an image as background and set it as clip path
 */
const loadBackground = (file) => {
  const reader = new FileReader();

  reader.onload = (event) => {
    const imgUrl = event.target.result;
    fabric.Image.fromURL(imgUrl, (img) => {
      // Calculate the center position
      var canvasCenterX = canvas.getWidth() / 2;
      var canvasCenterY = canvas.getHeight() / 2;
      // Set img attributes
      img.set({
        left: canvasCenterX - (img.width / 2),
        top: canvasCenterY - (img.height / 2),
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
