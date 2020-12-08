<template>
  <div>
    <video
      autoplay
      muted
      playsInline
      v-bind="camera"
      width="640"
      height="480"
      id="cam"
      style="position:fixed;top:50px"
    />
    <canvas
      id="c"
      width="640"
      height="480"
      style="position:fixed;top:50px"
    />
  </div>
</template>

<script>
import '@tensorflow/tfjs';
import * as cocoSsd from "@tensorflow-models/coco-ssd";
import { reactive, onMounted } from 'vue';

export default {
  name: 'App',
  setup(){
    const camera = reactive({
      srcObject : null,
    });

    const constraint = {
      audio : false,
      video : true
    }

    const webcam = async() => {
      try{
        camera.srcObject = await navigator.mediaDevices.getUserMedia(constraint);
        await camera.onloadedmetadata;
      }
      catch(e)
      {
        console.log(e);
      }
    }

    const load = async() => {
      try{
        await webcam();
        const loadModel = await cocoSsd.load({base:"mobilenet_v2"});
        const cam = document.getElementById('cam');
        detectFromVideoFrame(loadModel, cam);
      }
      catch(e)
      {
        console.log(e);
      }
    }

    const detectFromVideoFrame = (model, video) => {
      model.detect(video).then(predictions=>{
        showDetection(predictions);
        requestAnimationFrame(()=>{
          detectFromVideoFrame(model, video);
        });
      }, (error)=>{
        console.log(error);
      });
    }

    const showDetection = prediction => {
      const c = document.getElementById("c");
      const ctx = c.getContext("2d");
      ctx.clearRect(0,0, ctx.canvas.width, ctx.canvas.height );
      const font = "24px helvetica";
      ctx.font = font;
      ctx.textBaseline = "top";

      prediction.forEach(prediction=>{
        const x = prediction.bbox[0];
        const y = prediction.bbox[1];
        const width = prediction.bbox[2];
        const height = prediction.bbox[3];

        ctx.strokeStyle = "#2fff00";
        ctx.lineWidth = 1;
        ctx.strokeRect(x,y,width,height);

        ctx.fillStyle="#2fff00";  

        let predictionText = prediction.class;
        if(predictionText=="person")
        {
          predictionText="toilet";
        }

        const textWidth = ctx.measureText(predictionText).width;
        const textHeight = parseInt(font, 10);

        ctx.fillRect(x,y,textWidth+10, textHeight+10);
        ctx.fillRect(x,y+height-textHeight,textWidth+15, textHeight+10);

        ctx.fillStyle="#000000";
        ctx.fillText(predictionText,x,y);
        ctx.fillText(prediction.score.toFixed(2),x,y+height-textHeight);
      });
    }

    onMounted(()=>{
      if(navigator.mediaDevices.getUserMedia||navigator.mediaDevices.webkitGetUserMedia)
      {
        load();
      }else{
        alert("not supported");
      }
    });

    return{
      camera,
    }
  }
}
</script>


