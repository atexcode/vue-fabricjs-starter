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
        <el-upload :show-file-list="false" accept="image/*" :before-upload="onFileChange">
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
          <el-button icon="PictureFilled">BG Mask Image</el-button>
        </el-upload>
      </el-col>
    </el-row>


    <!-- Canvas -->
    <div>
      <canvas id="fabricCanvas"></canvas>
    </div>
  </div>
</template>

<script setup>
import { nextTick, onMounted, ref, watch } from 'vue';
import { fabric } from 'fabric';
import { trace } from 'potrace';

const color = ref('#000000');
const itemBackgroundColor = ref('#ffffff');

const activeObject = ref(null);
let canvas;
let shirtImage;
// let boundariesPath;


// Initialize Fabric.js canvas on component mount
onMounted(async () => {
  canvas = new fabric.Canvas('fabricCanvas', {
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

  canvas.on('selection:cleared', () => {
    activeObject.value = null;
  });


  //Keydown Handlers
  window.addEventListener('keydown', handleKeyDown);
});

// Function to add text to the canvas
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

// Function to add itext to the canvas
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

// Function to add image to the canvas
const onFileChange = (file) => {
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


// Set active object color
const setActiveColor = () => {
  console.log(activeObject.value);
  if (activeObject.value) {
    activeObject.value.set({ fill: color.value });
    canvas.renderAll();
  }
};

// Set background color of the canvas
const setBackgroundColor = () => {
  // canvas.setBackgroundColor(backgroundColor.value, canvas.renderAll.bind(canvas));
  if (activeObject.value) {
    activeObject.value.set({ backgroundColor: itemBackgroundColor.value });
    canvas.renderAll();
  }
};

// Remove the selected object when 'Delete' or 'Backspace' key is pressed
const eraseItem = () => {
  const obj = activeObject.value;  // Get the raw value of activeObject

  if (obj) {
    console.log("Object before removal:", obj); // Debugging log
    canvas.remove(canvas.getActiveObject());  // Directly remove the object
    canvas.renderAll();   // Rerender the canvas after removal

    console.log("Object removed successfully");
  } else {
    console.log("No active object found");
  }
};


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
      shirtImage = img;
      canvas.requestRenderAll();
    });
  };

  reader.onerror = (error) => {
    console.error("FileReader error:", error);
  };

  reader.readAsDataURL(file.raw || file);

  return false;
}

//Load Sample Canvas Image
// const loadSampleImage = () => {
//   console.log("Button clicked:before");
//   new fabric.Image.fromURL('./shirt2.png', (img) => {
//     console.log("Image Loaded: ");
//     console.log(img);

//     // Calculate the center position
//     var canvasCenterX = canvas.getWidth() / 2;
//     var canvasCenterY = canvas.getHeight() / 2;

//     img.set({
//       left: canvasCenterX - (img.width / 2),
//       top: canvasCenterY - (img.height / 2),
//       selectable: false,
//     });

//     img.sendToBack();
//     canvas.add(img);
//     canvas.clipPath = img;
//     shirtImage = img;

//   });

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
//Get Pixel Data



// function getBoundaryPathFromImage(imgElement) {
//   const tempCanvas = document.createElement('canvas');
//   const context = tempCanvas.getContext('2d');
//   tempCanvas.width = imgElement.width;
//   tempCanvas.height = imgElement.height;

//   // Draw the image on the temporary canvas
//   context.drawImage(imgElement, 0, 0);

//   // Get the image data (all pixels)
//   const imageData = context.getImageData(0, 0, imgElement.width, imgElement.height);
//   const data = imageData.data;

//   let pathData = 'M';
//   let started = false;

//   // Loop through the pixel data to find the boundary
//   for (let y = 0; y < imgElement.height; y++) {
//     for (let x = 0; x < imgElement.width; x++) {
//       const alpha = data[(y * imgElement.width + x) * 4 + 3]; // Get the alpha value

//       if (alpha > 0) { // If pixel is not transparent
//         if (!started) {
//           pathData += ` ${x} ${y}`;
//           started = true;
//         } else {
//           pathData += ` L ${x} ${y}`;
//         }
//         break; // Stop when the first opaque pixel is found
//       }
//     }
//   }

//   pathData += ' Z'; // Close the path
//   return pathData;
// }



//keydown definition
const handleKeyDown = (e) => {
  if (e.key === 'Delete') {
    eraseItem();
  } else if (e.key === 'O' || e.key === 'o') {

    // boundariesPath.set({
    //   width: shirtImage.width, // Force width to match the shirt image
    //   height: shirtImage.height, // Force height to match the shirt image
    //   scaleX: 1, // Ensure it's scaled correctly
    //   scaleY: 1,
    //   left: 0, // Align to top-left of the image
    //   top: 0,
    //   originX: 'left',
    //   originY: 'top',
    // });

    // // Apply the clipPath to the canvas
    // canvas.clipPath = boundariesPath;

    // console.log('Shirt Image Dimensions:', shirtImage.getBoundingRect());
    // console.log('Clip Path Dimensions:', boundariesPath.getBoundingRect());

    // console.log(boundariesPath);
    // canvas.bringForward(shirtImage);
    // canvas.clipPath = boundariesPath;
    // canvas.requestRenderAll();
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
