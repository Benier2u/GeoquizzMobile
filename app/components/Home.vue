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
      <Label v-if="estCompleteListe" class="h3" text="Choisissez une série"  style="text-align: center"></Label> 
      <ListPicker :items="s" v-model="index" v-if="estCompleteListe"/>
      <ActivityIndicator v-else color="green" busy="true" style="margin-top : 45%"></ActivityIndicator>
      <Button  v-if="estCompleteListe" class="btn btn-primary btn-rounded-lg" text="Ajouter une photo" @tap="takePicture"/>
      <button  v-if="estCompleteListe" class="btn btn-primary btn-rounded-lg" text="créer une série" @tap="nextPage"/>
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
import CryptoJS from "crypto-js";
import * as enums from "ui/enums";
import {
  ImageSource,
  fromFile,
  fromResource,
  fromBase64
} from "tns-core-modules/image-source";

const Serie = {
  template: `
    <page>
        <ActionBar title="Geoquizz">
          <NavigationButton android.systemIcon="ic_menu_back" @tap="$navigateBack"/>
        </ActionBar>
      <StackLayout>
        <Label class="h2" text=" nom de la ville" style="margin-top:2%"></Label> 
        <text-field v-model="ville"></text-field>
        <Label class="h2" text="longitude" style="margin-top:2%"></Label> 
        <text-field v-model="longitude"></text-field>
        <Label class="h2" text="latitude" style="margin-top:2%"></Label> 
        <text-field v-model="latitude"></text-field>
        <button  v-bind:isEnabled="this.ville !== '' && this.longitude!== '' && this.latitude !== ''"class="btn btn-primary btn-rounded-lg" text="creer" @tap="createSerie"/>
      </StackLayout>
  </page>
  `,
  data() {
    return {
      ville: "",
      longitude: "",
      latitude: ""
    };
  },
  methods: {
    createSerie() {
      let that = this;
      axios
        .post(
          "https://mobile-geoquizzatelier.pagekite.me/series",
          {
            ville: that.ville,
            map_refs: that.longitude + " " + that.latitude,
            dist: 1
          },
          {
            headers: {
              "Content-Type": "application/json"
            }
          }
        )
        .then(response => {
          this.$navigateBack();
        })
        .catch(e => {
          alert(e);
        });
    }
  }
};

export default {
  data() {
    return {
      images: [],
      latitude: "",
      longitude: "",
      config: config.address,
      s: [],
      index: 0,
      estCompleteListe: false
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
              console.log(img.src);
              this.images.push(img);
              let formdata = new FormData();
              let source = new ImageSource();
              source.fromAsset(imageAsset).then(source => {
                let base64image = source.toBase64String("jpeg", 60);
                base64image = "data:image/jpeg;base64," + base64image;
                formdata.append("file", base64image);
                that.getLocation();

                let timestamp = ((Date.now() / 1000) | 0).toString();
                let api_key = "742671633714977";
                let api_secret = "SR4i2HyTa2MK9-HDSCEZz7a4fPg";
                let cloud = "dnymyg55r";
                let hash_string = "timestamp=" + timestamp + api_secret;
                let signature = CryptoJS.SHA1(hash_string).toString();
                let upload_url =
                  "https://api.cloudinary.com/v1_1/" + cloud + "/image/upload";

                let xhr = new XMLHttpRequest();
                xhr.open("POST", upload_url);
                xhr.onload = () => {
                  console.log(xhr.responseText);
                  let x = JSON.parse(xhr.responseText);
                  axios
                    .get(this.config + "series/")
                    .then(response => {
                      var id = response.data[that.index].id;
                      axios
                        .post(this.config + "series/" + id + "/photos", {
                          description: "",
                          position: that.latitude + " " + that.longitude,
                          id: x.public_id
                        })
                        .then(response => {})

                        .catch(e => {
                          this.errors.push(e);
                        });
                    })
                    .catch(e => {
                      this.errors.push(e);
                    });
                };
                formdata.append("timestamp", timestamp);
                formdata.append("api_key", api_key);
                formdata.append("signature", signature);
                xhr.send(formdata);
              });
            })
            .catch(e => {
              console.log("error:", e);
              alert("error");
            });
        })
        .catch(e => {
          console.log("Error requesting permission");
        });
    },

    nextPage() {
      this.$navigateTo(Serie);
    }
  },
  created() {
    axios.get(this.config + "series").then(response => {
      for (let i = 0; i < response.data.length; i++) {
        this.s.push(response.data[i].ville);
      }
      this.estCompleteListe = true;
      this.s.refresh();
    });
  }
};
</script>
