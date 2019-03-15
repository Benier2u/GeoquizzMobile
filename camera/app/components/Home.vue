<template>
  <Page>
    <StackLayout>
      <Button text="Take Picture" @tap="takePicture"/>
      <Image :src="img" width="75" height="75"/>
    </StackLayout>
  </Page>
</template>

<template>
  <Page>
    <ActionBar title="Geoquizz"/>
    <StackLayout>
      <Button class="btn btn-primary btn-rounded-lg" text="Take Picture" @tap="takePicture"/>
      <button
        class="btn btn-primary btn-rounded-lg"
        text="créer une série"
        @tap="$navigateTo(Serie)"
      />
      <Button
        class="btn btn-primary btn-rounded-lg"
        text="Get Current Location "
        col="1"
        textWrap="true"
        @tap="getLocation"
      />
      <text-field v-model="latitude"></text-field>
      <text-field v-model="longitude"></text-field>
      <text-field v-model="config"></text-field>
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
import config from "../../config.json";
import axios from "axios";
import { Image } from "tns-core-modules/ui/image";
import {
  TNSHttpFormData,
  TNSHttpFormDataParam,
  TNSHttpFormDataResponse
} from "nativescript-http-formdata";

const Serie = {
  template: `
    <page>
        <ActionBar title="Geoquizz"/>
      <StackLayout>
        <text-field v-mondel="ville"></text-field>
        <button class="btn btn-primary btn-rounded-lg" text="valider" @tap=$navigateBack />
        <button class="btn btn-primary btn-rounded-lg" text="creer" @tap="createSerie" />
      </StackLayout>
  </page>
  `
};

export default {
  data() {
    return {
      images: [],
      latitude: "",
      longitude: "",
      config: config.address,
      ville: ""
    };
  },
  methods: {
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

      axios
        .get(this.config + "series/")
        .then(response => {
          var id = response.data[0].id;
          axios
            .post(this.config + "series/" + id + "/photos", {
              description: "lol",
              position: that.latitude + " " + that.longitude
            })
            .then(response => {
              that.images[0].src = response.data + ".jpg";
              let formData = new FormData();
              formData.append("image", that.images[0]);
              axios
                .post(this.config + "images/upload", {
                  headers: {
                    "Content-Type": "multipart/form-data"
                  },
                  data: formData
                })
                .then(response => {
                  alert("fdp");
                })
                .catch(e => {
                  this.errors.push(e);
                  alert("gros con");
                });
            })
            .catch(e => {
              this.errors.push(e);
            });
        })
        .catch(e => {
          this.errors.push(e);
        });
    },

    takePicture() {
      var that = this;
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
              that.getLocation();
            })
            .catch(e => {
              console.log("error:", e);
            });
        })
        .catch(e => {
          console.log("Error requesting permission");
        });
    },

    createSerie() {
      let that = this;
      axios
        .post(this.config + "/series", {
          ville: that.ville,
          map_refs: "48.68 6.16",
          Dist: 1
        })
        .then(response => {
          alert(creation);
        })
        .catch(e => {
          alert("error");
        });
    }
  }
};
</script>
