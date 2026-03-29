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
      <el-col :span="1.5">
        <el-button :type="isDrawingMode ? 'primary' : ''" @click="toggleDrawingMode">
          {{ isDrawingMode ? 'Stop Drawing' : 'Draw' }}
        </el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="addRectangle"> Rectangle</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="addCircle"> Circle</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="addTriangle"> Triangle</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="addLine"> Line</el-button>
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
      <el-col :span="1.5">
        <el-button icon="ArrowDown" :disabled="!history.hasUndo" @click="canvas.undo()"> Undo</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button icon="ArrowUp" :disabled="!history.hasRedo" @click="canvas.redo()"> Redo</el-button>
      </el-col>
    </el-row>

    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content;">
      <el-col :span="1.5">
        <el-input v-model="svgStrURL" placeholder="SVG URL"></el-input>
      </el-col>
      <el-col :span="1.5">
        <el-button icon="PictureFilled" @click="InsertSvg">Load SVG</el-button>
      </el-col>
      <el-col :span="1.5">
        <span style="line-height: 32px; padding: 0 10px;">Canvas BG:</span>
      </el-col>
      <el-col :span="1.5">
        <el-color-picker v-model="canvasBackgroundColor" @change="changeCanvasBackground" />
      </el-col>
    </el-row>

    <!-- Zoom Controls -->
    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content; margin-top: 10px;">
      <el-col :span="1.5">
        <el-button icon="ZoomIn" @click="zoomIn"> Zoom In</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button icon="ZoomOut" @click="zoomOut"> Zoom Out</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="resetZoom"> 100%</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="fitToScreen"> Fit</el-button>
      </el-col>
      <el-col :span="1.5">
        <span style="line-height: 32px; padding: 0 10px;">{{ zoomLevel }}%</span>
      </el-col>
    </el-row>

    <!-- Export Controls -->
    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content; margin-top: 10px;">
      <el-col :span="1.5">
        <el-button icon="Download" @click="exportAsPNG"> Export PNG</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button icon="Download" @click="exportAsJPG"> Export JPG</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button icon="Download" @click="saveAsJSON"> Save JSON</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-upload :show-file-list="false" accept=".json" :before-upload="loadFromJSON">
          <el-button icon="Upload"> Load JSON</el-button>
        </el-upload>
      </el-col>
    </el-row>

    <!-- Alignment Controls -->
    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content; margin-top: 10px;" v-if="activeObject">
      <el-col :span="1.5">
        <el-button @click="alignLeft"> Align Left</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="alignCenterH"> Center H</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="alignRight"> Align Right</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="alignTop"> Align Top</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="alignCenterV"> Center V</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="alignBottom"> Align Bottom</el-button>
      </el-col>
    </el-row>

    <!-- Z-Index Controls -->
    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content; margin-top: 10px;" v-if="activeObject">
      <el-col :span="1.5">
        <el-button @click="bringToFront"> To Front</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="bringForward"> Forward</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="sendBackward"> Backward</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="sendToBack"> To Back</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="flipHorizontal"> Flip H</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="flipVertical"> Flip V</el-button>
      </el-col>
      <el-col :span="1.5">
        <span style="line-height: 32px; padding: 0 10px;">Opacity:</span>
      </el-col>
      <el-col :span="1.5">
        <el-slider v-model="objectOpacity" @change="changeOpacity" :min="0" :max="1" :step="0.1" style="width: 150px;" />
      </el-col>
    </el-row>

    <!-- Text Formatting Controls -->
    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content; margin-top: 10px;" v-if="activeObject && (activeObject.type === 'textbox' || activeObject.type === 'i-text' || activeObject.type === 'text')">
      <el-col :span="1.5">
        <el-select v-model="selectedFont" @change="changeFont" placeholder="Font">
          <el-option label="Arial" value="Arial"></el-option>
          <el-option label="Helvetica" value="Helvetica"></el-option>
          <el-option label="Times New Roman" value="Times New Roman"></el-option>
          <el-option label="Courier New" value="Courier New"></el-option>
          <el-option label="Georgia" value="Georgia"></el-option>
          <el-option label="Verdana" value="Verdana"></el-option>
          <el-option label="Comic Sans MS" value="Comic Sans MS"></el-option>
        </el-select>
      </el-col>
      <el-col :span="1.5">
        <el-input-number v-model="fontSize" @change="changeFontSize" :min="8" :max="200" />
      </el-col>
      <el-col :span="1.5">
        <el-button @click="toggleBold"><strong>B</strong></el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="toggleItalic"><em>I</em></el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="toggleUnderline"><u>U</u></el-button>
      </el-col>
    </el-row>

    <!-- Image Filter Controls -->
    <el-row class="toolbar bg-group ms-2" :gutter="5" style="width: fit-content; margin-top: 10px;" v-if="activeObject && activeObject.type === 'image'">
      <el-col :span="1.5">
        <el-button @click="applyGrayscale"> Grayscale</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="applySepia"> Sepia</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="applyBlur"> Blur</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="applyBrightness(0.2)"> Brightness+</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="applyBrightness(-0.2)"> Brightness-</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="applyContrast(0.2)"> Contrast+</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="applyContrast(-0.2)"> Contrast-</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button @click="clearFilters"> Clear Filters</el-button>
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
import 'fabric-history';

const color = ref('#000000');
const itemBackgroundColor = ref('#ffffff');
const zoomLevel = ref(1);
const isDrawingMode = ref(false);
const selectedFont = ref('Helvetica');
const fontSize = ref(20);
const objectOpacity = ref(1);
const canvasBackgroundColor = ref('#ffffff');

const activeObject = ref(null);
const history = ref({
  hasUndo: false,
  hasRedo: false,
});
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

  canvas.on('history:append', () => {
    console.log("History appended");
    setHistoryFlags();
  });

  canvas.on('history:undo', () => {
    console.log("History undo");
    setHistoryFlags();
  });

  canvas.on('history:redo', () => {
    console.log("History redo");
    setHistoryFlags();
  });

  canvas.on('history:clear', () => {
    console.log("History cleared");
    setHistoryFlags();
  });


  //Keydown Handlers
  window.addEventListener('keydown', handleKeyDown);
});

/**
 * Method to set flag variable values for undo and redo
 */
const setHistoryFlags = () => {
  if (!canvas) return;
  history.value.hasUndo = canvas.canUndo();
  history.value.hasRedo = canvas.canRedo();
}


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

  // Make the items to be resized from its center
  fabric.Object.prototype.centeredScaling = true;
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
 * Duplicate the selected object
 */
const duplicateObject = () => {
  const obj = activeObject.value;

  if (obj) {
    obj.clone((cloned) => {
      cloned.set({
        left: cloned.left + 10,
        top: cloned.top + 10,
      });
      canvas.add(cloned);
      canvas.setActiveObject(cloned);
      canvas.renderAll();
    });
  }
};

/**
 * Add Rectangle shape to canvas
 */
const addRectangle = () => {
  const canvasCenter = getCenter();
  const rect = new fabric.Rect({
    left: canvasCenter.x - 50,
    top: canvasCenter.y - 50,
    width: 100,
    height: 100,
    fill: itemBackgroundColor.value,
    stroke: color.value,
    strokeWidth: 2,
  });
  canvas.add(rect);
};

/**
 * Add Circle shape to canvas
 */
const addCircle = () => {
  const canvasCenter = getCenter();
  const circle = new fabric.Circle({
    left: canvasCenter.x - 50,
    top: canvasCenter.y - 50,
    radius: 50,
    fill: itemBackgroundColor.value,
    stroke: color.value,
    strokeWidth: 2,
  });
  canvas.add(circle);
};

/**
 * Add Triangle shape to canvas
 */
const addTriangle = () => {
  const canvasCenter = getCenter();
  const triangle = new fabric.Triangle({
    left: canvasCenter.x - 50,
    top: canvasCenter.y - 50,
    width: 100,
    height: 100,
    fill: itemBackgroundColor.value,
    stroke: color.value,
    strokeWidth: 2,
  });
  canvas.add(triangle);
};

/**
 * Add Line shape to canvas
 */
const addLine = () => {
  const canvasCenter = getCenter();
  const line = new fabric.Line([canvasCenter.x - 50, canvasCenter.y, canvasCenter.x + 50, canvasCenter.y], {
    stroke: color.value,
    strokeWidth: 2,
  });
  canvas.add(line);
};

/**
 * Zoom In
 */
const zoomIn = () => {
  let zoom = canvas.getZoom();
  zoom += 0.1;
  if (zoom > 3) zoom = 3; // Max zoom level
  canvas.setZoom(zoom);
  zoomLevel.value = Math.round(zoom * 100);
  canvas.renderAll();
};

/**
 * Zoom Out
 */
const zoomOut = () => {
  let zoom = canvas.getZoom();
  zoom -= 0.1;
  if (zoom < 0.1) zoom = 0.1; // Min zoom level
  canvas.setZoom(zoom);
  zoomLevel.value = Math.round(zoom * 100);
  canvas.renderAll();
};

/**
 * Reset Zoom to 100%
 */
const resetZoom = () => {
  canvas.setZoom(1);
  zoomLevel.value = 100;
  canvas.renderAll();
};

/**
 * Fit canvas to viewport
 */
const fitToScreen = () => {
  const canvasWidth = canvas.getWidth();
  const canvasHeight = canvas.getHeight();
  const windowWidth = window.innerWidth - 100;
  const windowHeight = window.innerHeight - 200;

  const scaleX = windowWidth / canvasWidth;
  const scaleY = windowHeight / canvasHeight;
  const scale = Math.min(scaleX, scaleY, 1);

  canvas.setZoom(scale);
  zoomLevel.value = Math.round(scale * 100);
  canvas.renderAll();
};

/**
 * Export canvas as PNG
 */
const exportAsPNG = () => {
  const dataURL = canvas.toDataURL({
    format: 'png',
    quality: 1,
    multiplier: 2, // Higher resolution
  });
  downloadImage(dataURL, 'canvas-export.png');
};

/**
 * Export canvas as JPG
 */
const exportAsJPG = () => {
  const dataURL = canvas.toDataURL({
    format: 'jpeg',
    quality: 0.9,
    multiplier: 2, // Higher resolution
  });
  downloadImage(dataURL, 'canvas-export.jpg');
};

/**
 * Helper function to download image
 */
const downloadImage = (dataURL, filename) => {
  const link = document.createElement('a');
  link.href = dataURL;
  link.download = filename;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};

/**
 * Align selected object to left
 */
const alignLeft = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ left: 0 });
    canvas.renderAll();
  }
};

/**
 * Align selected object to center horizontally
 */
const alignCenterH = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ left: (canvas.getWidth() - obj.width * obj.scaleX) / 2 });
    canvas.renderAll();
  }
};

/**
 * Align selected object to right
 */
const alignRight = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ left: canvas.getWidth() - obj.width * obj.scaleX });
    canvas.renderAll();
  }
};

/**
 * Align selected object to top
 */
const alignTop = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ top: 0 });
    canvas.renderAll();
  }
};

/**
 * Align selected object to center vertically
 */
const alignCenterV = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ top: (canvas.getHeight() - obj.height * obj.scaleY) / 2 });
    canvas.renderAll();
  }
};

/**
 * Align selected object to bottom
 */
const alignBottom = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ top: canvas.getHeight() - obj.height * obj.scaleY });
    canvas.renderAll();
  }
};

/**
 * Toggle drawing mode for freehand drawing
 */
const toggleDrawingMode = () => {
  isDrawingMode.value = !isDrawingMode.value;
  canvas.isDrawingMode = isDrawingMode.value;

  if (isDrawingMode.value) {
    canvas.freeDrawingBrush.color = color.value;
    canvas.freeDrawingBrush.width = 3;
  }
};

/**
 * Update brush color when color changes
 */
watch(() => color.value, () => {
  if (canvas && canvas.freeDrawingBrush) {
    canvas.freeDrawingBrush.color = color.value;
  }
  setActiveColor();
});

/**
 * Toggle bold formatting for selected text
 */
const toggleBold = () => {
  const obj = activeObject.value;
  if (obj && (obj.type === 'textbox' || obj.type === 'i-text' || obj.type === 'text')) {
    const currentWeight = obj.fontWeight;
    obj.set({ fontWeight: currentWeight === 'bold' ? 'normal' : 'bold' });
    canvas.renderAll();
  }
};

/**
 * Toggle italic formatting for selected text
 */
const toggleItalic = () => {
  const obj = activeObject.value;
  if (obj && (obj.type === 'textbox' || obj.type === 'i-text' || obj.type === 'text')) {
    const currentStyle = obj.fontStyle;
    obj.set({ fontStyle: currentStyle === 'italic' ? 'normal' : 'italic' });
    canvas.renderAll();
  }
};

/**
 * Toggle underline for selected text
 */
const toggleUnderline = () => {
  const obj = activeObject.value;
  if (obj && (obj.type === 'textbox' || obj.type === 'i-text' || obj.type === 'text')) {
    obj.set({ underline: !obj.underline });
    canvas.renderAll();
  }
};

/**
 * Change font family for selected text
 */
const changeFont = () => {
  const obj = activeObject.value;
  if (obj && (obj.type === 'textbox' || obj.type === 'i-text' || obj.type === 'text')) {
    obj.set({ fontFamily: selectedFont.value });
    canvas.renderAll();
  }
};

/**
 * Change font size for selected text
 */
const changeFontSize = () => {
  const obj = activeObject.value;
  if (obj && (obj.type === 'textbox' || obj.type === 'i-text' || obj.type === 'text')) {
    obj.set({ fontSize: fontSize.value });
    canvas.renderAll();
  }
};

/**
 * Save canvas as JSON
 */
const saveAsJSON = () => {
  const json = JSON.stringify(canvas.toJSON());
  const blob = new Blob([json], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = 'canvas-design.json';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(url);
};

/**
 * Load canvas from JSON file
 */
const loadFromJSON = (file) => {
  const reader = new FileReader();

  reader.onload = (event) => {
    const json = event.target.result;
    canvas.loadFromJSON(json, () => {
      canvas.renderAll();
      console.log('Canvas loaded from JSON successfully');
    });
  };

  reader.onerror = (error) => {
    console.error('FileReader error:', error);
  };

  reader.readAsText(file.raw || file);
  return false;
};

/**
 * Bring selected object to front
 */
const bringToFront = () => {
  const obj = activeObject.value;
  if (obj) {
    canvas.bringToFront(obj);
    canvas.renderAll();
  }
};

/**
 * Send selected object to back
 */
const sendToBack = () => {
  const obj = activeObject.value;
  if (obj) {
    canvas.sendToBack(obj);
    canvas.renderAll();
  }
};

/**
 * Bring selected object forward
 */
const bringForward = () => {
  const obj = activeObject.value;
  if (obj) {
    canvas.bringForward(obj);
    canvas.renderAll();
  }
};

/**
 * Send selected object backward
 */
const sendBackward = () => {
  const obj = activeObject.value;
  if (obj) {
    canvas.sendBackward(obj);
    canvas.renderAll();
  }
};

/**
 * Change opacity of selected object
 */
const changeOpacity = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ opacity: objectOpacity.value });
    canvas.renderAll();
  }
};

/**
 * Change canvas background color
 */
const changeCanvasBackground = () => {
  canvas.backgroundColor = canvasBackgroundColor.value;
  canvas.renderAll();
};

/**
 * Flip selected object horizontally
 */
const flipHorizontal = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ flipX: !obj.flipX });
    canvas.renderAll();
  }
};

/**
 * Flip selected object vertically
 */
const flipVertical = () => {
  const obj = activeObject.value;
  if (obj) {
    obj.set({ flipY: !obj.flipY });
    canvas.renderAll();
  }
};

/**
 * Apply grayscale filter to selected image
 */
const applyGrayscale = () => {
  const obj = activeObject.value;
  if (obj && obj.type === 'image') {
    const filter = new fabric.Image.filters.Grayscale();
    obj.filters.push(filter);
    obj.applyFilters();
    canvas.renderAll();
  }
};

/**
 * Apply sepia filter to selected image
 */
const applySepia = () => {
  const obj = activeObject.value;
  if (obj && obj.type === 'image') {
    const filter = new fabric.Image.filters.Sepia();
    obj.filters.push(filter);
    obj.applyFilters();
    canvas.renderAll();
  }
};

/**
 * Apply blur filter to selected image
 */
const applyBlur = () => {
  const obj = activeObject.value;
  if (obj && obj.type === 'image') {
    const filter = new fabric.Image.filters.Blur({ blur: 0.5 });
    obj.filters.push(filter);
    obj.applyFilters();
    canvas.renderAll();
  }
};

/**
 * Apply brightness filter to selected image
 */
const applyBrightness = (value = 0.2) => {
  const obj = activeObject.value;
  if (obj && obj.type === 'image') {
    const filter = new fabric.Image.filters.Brightness({ brightness: value });
    obj.filters.push(filter);
    obj.applyFilters();
    canvas.renderAll();
  }
};

/**
 * Apply contrast filter to selected image
 */
const applyContrast = (value = 0.2) => {
  const obj = activeObject.value;
  if (obj && obj.type === 'image') {
    const filter = new fabric.Image.filters.Contrast({ contrast: value });
    obj.filters.push(filter);
    obj.applyFilters();
    canvas.renderAll();
  }
};

/**
 * Clear all filters from selected image
 */
const clearFilters = () => {
  const obj = activeObject.value;
  if (obj && obj.type === 'image') {
    obj.filters = [];
    obj.applyFilters();
    canvas.renderAll();
  }
};

/**
 * Update opacity slider when object is selected
 */
watch(() => activeObject.value, (newObj) => {
  if (newObj) {
    objectOpacity.value = newObj.opacity ?? 1;
    if (newObj.type === 'textbox' || newObj.type === 'i-text' || newObj.type === 'text') {
      selectedFont.value = newObj.fontFamily || 'Helvetica';
      fontSize.value = newObj.fontSize || 20;
    }
  }
});

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

      canvas.add(img);
      canvas.clipPath = img;
      // Send Img to Back
      img.sendToBack();
      canvas.requestRenderAll();
    });
  };

  reader.onerror = (error) => {
    console.error("FileReader error:", error);
  };

  reader.readAsDataURL(file.raw || file);

  return false;
}

const svgStrURL = ref('');

/**
 * Load SVG from URL
 */
const InsertSvg = () => {
  fabric.loadSVGFromURL(svgStrURL.value, function (objects, options) {
    let svg = fabric.util.groupSVGElements(objects, options);

    // Apply saved properties
    svg.set({
        left: 130,
        top: 30,
    });

    canvas.add(svg);
    
    // Re-Render the canvas
    canvas.renderAll();
  });
}

loadControlls();

//keydown definition
const handleKeyDown = (e) => {
  // Delete key - remove selected object
  if (e.key === 'Delete' || e.key === 'Backspace') {
    eraseItem();
  }
  // Ctrl+Z - Undo
  else if (e.ctrlKey && e.key === 'z') {
    e.preventDefault();
    if (canvas.canUndo()) {
      canvas.undo();
    }
  }
  // Ctrl+Y or Ctrl+Shift+Z - Redo
  else if ((e.ctrlKey && e.key === 'y') || (e.ctrlKey && e.shiftKey && e.key === 'z')) {
    e.preventDefault();
    if (canvas.canRedo()) {
      canvas.redo();
    }
  }
  // Ctrl+D - Duplicate selected object
  else if (e.ctrlKey && e.key === 'd') {
    e.preventDefault();
    duplicateObject();
  }
};

watch(
  () => itemBackgroundColor.value,
  () => setBackgroundColor()
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

.bg-group {
  background-color: #f0f0f0;
  padding: 10px;
}

.ms-1 {
  margin-left: 0.3rem !important;
}
</style>
