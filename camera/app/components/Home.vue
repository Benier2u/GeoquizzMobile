<template>
  <Page>
    <ActionBar title="Camera Tests FTW!"/>
    <StackLayout>
      <Button text="Take Picture" @tap="takePicture"/>
      <Image :src="img" width="75" height="75"/>
    </StackLayout>
  </Page>
</template>

<template>
  <Page>
    <ActionBar title="Camera Tests FTW!"/>
    <StackLayout>
      <Button text="Take Picture" @tap="takePicture"/>
      <Button text="Choose Picture" @tap="selectPicture"/>
      <Button text="Get Current Location" col="1" textWrap="true" @tap="getLocation"/>
      <!-- <text-field v-model="position.longitude"></text-field> -->
      <text-field v-model="latitude"></text-field>
      <text-field v-model="longitude"></text-field>
      <WrapLayout>
        <Image v-for="img in images" :src="img.src" width="75" height="75"/>
      </WrapLayout>
    </StackLayout>
  </Page>
</template>

<script>
import * as camera from "nativescript-camera";
import * as imagepicker from "nativescript-imagepicker";
import * as geolocation from "nativescript-geolocation";
import { Accuracy } from "tns-core-modules/ui/enums";

import { Image } from "tns-core-modules/ui/image";

export default {
  data() {
    return {
			images:[],
			latitude:"",
			longitude:"",
    };
  },
  methods: {
    selectPicture() {
      let context = imagepicker.create({
        mode: "multiple"
      });

      context
        .authorize()
        .then(function() {
          return context.present();
        })
        .then(selection => {
          selection.forEach(selected => {
            console.log(JSON.stringify(selected));

            let img = new Image();
            img.src = selected;
            this.images.push(img);
          });
        })
        .catch(function(e) {
          console.log("error in selectPicture", e);
        });
    },
    getLocation() {
      let that = this;
      geolocation
        .getCurrentLocation({
          desiredAccuracy: Accuracy.high,
          maximumAge: 5000,
          timeout: 10000
        })
        .then(
          function(loc) {
            if (loc) {
							that.latitude = loc.latitude;
							that.longitude = loc.longitude;
            }
          },
          function(e) {
            console.log("Error: " + (e.message || e));
          }
        );
    },
    takePicture() {
      camera
        .requestPermissions()
        .then(() => {
          camera
            .takePicture({
              width: 300,
              height: 300,
              keepAspectRatio: true,
              saveToGallery: false
            })
            .then(imageAsset => {
              let img = new Image();
              img.src = imageAsset;
              this.images.push(img);
              console.log("ive got " + this.images.length + " images now.");
            })
            .catch(e => {
              console.log("error:", e);
            });
        })
        .catch(e => {
          console.log("Error requesting permission");
        });
    }
  }
};
</script>
