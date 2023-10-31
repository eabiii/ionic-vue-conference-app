<template>
  <ion-page>
    <ion-header translucent="true">
        <ion-toolbar>
          <ion-title>Maps</ion-title>
          <ion-buttons slot="start">
            <ion-menu-button></ion-menu-button>
          </ion-buttons>
        </ion-toolbar>
  
      </ion-header>

        <capacitor-google-map
        ref="mapRef"
      >
      </capacitor-google-map>

    </ion-page>
  </template>
  
  <script lang="ts" setup>
 import { onMounted, nextTick, ref, onUnmounted } from "vue";
import { GoogleMap } from "@capacitor/google-maps";
import {
    IonPage,
    IonHeader,
    IonToolbar,
    IonButtons,
    IonMenuButton,

    IonContent,
    IonList,
    IonItem,
    IonTitle,
    IonGrid,
    IonRow,
    IonCol,
    IonLabel,
    IonCard,
    IonCardContent,
    IonCardHeader,
    IonAvatar,
  loadingController,
  } from '@ionic/vue';
import { ticket } from "ionicons/icons";
const props = defineProps<{
  markerData: { coordinate: any; title: string; snippet: string }[];
}>();
const emits = defineEmits<{
  (event: "onMarkerClicked", info: any): void;
  (event: "onMapClicked"): void;
}>();

const mapRef = ref<HTMLElement>();
const markerIds = ref<string[] | undefined>();
let newMap: GoogleMap;


onMounted(async () => {
  console.log("mounted ", mapRef.value);
  await nextTick();
  await createMap();
});

onUnmounted(() => {
  console.log("onunmounted");
  newMap.removeMarkers(markerIds?.value as string[]);
});


const addSomeMarkers = async (newMap: GoogleMap) => {
  markerIds?.value && newMap.removeMarkers(markerIds?.value as string[]);
  console.log("🚀 ~ file: Maps.vue:81 ~ markers ~ props.markerData:", props.markerData)
  const markers = [
    {
      coordinate:{
        lat:14.546683247434416, 
        lng:121.06432498060155
      },
      title:'localhostead'
    }
  ]
  // // Plot each point on the map
  // let markers = props.markerData.map(({ coordinate, title, snippet }) => {
  //   return {
  //     coordinate, 
  //     title,
  //     snippet,
  //   };
  // });

  await newMap.addMarkers(markers);
};

  const addCurrentLocationMarker = async (newMap: GoogleMap) => {
        const position = await getCurrentPosition();
        const marker = await newMap.addMarker({
          coordinate: position,
          title: 'Current Location',
          snippet: 'You are here',
        });
        console.log("🚀 ~ file: Maps.vue:102 ~ addCurrentLocationMarker ~ position:", position)
        markerIds.value = [...(markerIds.value || []), marker];
      };

    const getCurrentPosition = (): Promise<google.maps.LatLngLiteral> => {
      return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            resolve({
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            });
          },
          (error) => {
            reject(error);
          }
        );
      });
    };

async function createMap() {
  if (!mapRef.value) return;

  // render map using capacitor plugin
  newMap = await GoogleMap.create({
    id: "my-cool-map",
    element: mapRef.value,
    apiKey: 'GOOGLE_API_KEY',
    config: {
      center: {
        lat: 14.546683247434416,
        lng: 121.06432498060155,
      },
      zoom: 12,
    },
  });

  addSomeMarkers(newMap);
  addCurrentLocationMarker(newMap);


  newMap.setOnMarkerClickListener((event) => {
    emits("onMarkerClicked", event);
  });

  newMap.setOnMapClickListener(() => {
    console.log("map clicked")
    emits("onMapClicked");
  });
}
  </script>

  <style scoped>
  capacitor-google-map {
    display: inline-block;
  width: 100%;
  height: 100%;

}
ion-content::part(background) {
    background-color: transparent;
  }
    body {
  background: transparent;
}
    </style>