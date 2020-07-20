<template>
    <div class="row no-gutters">
        <div class="col-sm-3">
            <div class="toolbox">
                <div class="sticky-top bg-white shadow-sm p-2">
                    <div class="form-group d-flex">
                        <label for="cityName" class="mr-2 col-form-label text-right">県市</label>
                        <div class="flex-fill">
                            <select id="cityName" class="form-control" v-model="selectedCity" @change="chooseCity">
                                <option>{{ selectedCity }}</option>
                                <option v-for="item in cityName" :key="item.CityEngName" v-bind:value="item.CityName">{{ item.CityName }}</option>
                            </select>
                        </div>
                    </div>
                    
                    <div class="form-group d-flex">
                        <label for="area" class="mr-2 col-form-label text-right">区域</label>
                        <div class="flex-fill">
                            <select id="area" class="form-control" v-model="selectedTown" @change="chooseTown">
                                <option v-for="item in region" :key="item.AreaName">{{ item.AreaName }}</option>
                            </select>
                        </div>
                    </div>
                    <p class="mb-0 small text-muted text-right">（緑は十分貯まる）</p>
                </div>
                <ul class="list-group">
                    <a class="list-group-item text-left" v-for="item in localData" :key="item.properties.id" @click="penTo(item)">
                        <h3>{{ item.properties.name }}</h3>
                        <p class="mb-0">
                            成人マスク：{{ item.properties.mask_adult }} | 兒童マスク：{{ item.properties.mask_child }}
                        </p>
                        <p class="mb-0">アドレス：{{ item.properties.address }}
                            <a href="https://www.google.com.tw/maps/place/..." target="_blank" title="Google Map">            
                            </a>
                        </p>
                    </a>
                </ul>
            </div>
        </div>
        <div class="col-sm-9">
            <div id="map"></div>
        </div>
    </div>
</template>

<style lang="scss">
    @import "bootstrap/scss/bootstrap";

    #map {
        height: 100vh;
    }

    .highlight {
        background: #e9ffe3;
    }

    .toolbox {
        height: 100vh;
        overflow-y: auto;

        a {
            cursor: pointer;
        }
    }
</style>

<script>
    import L from "leaflet";
    import cityName from "./assets/cityName.json";

    let osmMap ={}
    const iconsConfig = {
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      }
    const icons = {
        green: new L.Icon({
          iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
          ...iconsConfig,
        }),
        grey: new L.Icon({
          iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-grey.png',
          ...iconsConfig,
        }),
      };

    export default {
        name: "App",
        components: {},
        data: () => ({
            data: [],
            cityName,
            region:[],
            selectedCity:'please choose',
            selectedTown:'',
            localData:[],
        }),
        methods: {
            chooseCity(){
                
                this.cityName.forEach((value) => {
                    if(value.CityName == this.selectedCity){
                        this.region = value.AreaList
                    }
                })
            },
            chooseTown(){
                console.log(osmMap);
                osmMap.eachLayer((layer)=>{
                    if (layer instanceof L.Marker) {
                        osmMap.removeLayer(layer);
                      }
                })
                this.data.forEach((value) => {
                    console.log(value)
                    if(value.properties.town == this.selectedTown){
                        const icon = value.properties.mask_adult || value.properties.mask_child ? icons.green : icons.grey;
                        osmMap.panTo(new L.LatLng(value.geometry.coordinates[1],value.geometry.coordinates[0]));
                        this.localData.push(value);
                        L.marker([value.geometry.coordinates[1],value.geometry.coordinates[0]],{icon}).addTo(osmMap).bindPopup(`<strong>${value.properties.name}</strong> <br>
                        マスク残る：<strong>成人 - ${value.properties.mask_adult ? `${value.properties.mask_adult} 枚` : '未取得資料'}/ 兒童 - ${value.properties.mask_child ? `${value.properties.mask_child} 枚` : '未取得資料'}</strong><br>
                       `).openPopup();
                    }
                });
       
               
            },
            penTo(item){
                const icon = item.properties.mask_adult || item.properties.mask_child ? icons.green : icons.grey;
                osmMap.panTo(new L.LatLng(item.geometry.coordinates[1],item.geometry.coordinates[0]));
                L.marker([item.geometry.coordinates[1],item.geometry.coordinates[0]],{icon}).addTo(osmMap).bindPopup(`<strong>${item.properties.name}</strong> <br>
                        マスク残る：<strong>成人 - ${item.properties.mask_adult ? `${item.properties.mask_adult} 枚` : '未取得資料'}/ 兒童 - ${item.properties.mask_child ? `${item.properties.mask_child} 枚` : '未取得資料'}</strong><br>
                       `).openPopup();
            },
        },
        mounted() {
            osmMap = L.map('map', {
                center: [25.03, 121.55],
                zoom: 15,
              });

          L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '<a target="_blank" href="https://www.openstreetmap.org/">© OpenStreetMap 貢獻者</a>',
            maxZoom: 18,
          }).addTo(osmMap);

            const url = "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
            this.$http.get(url).then(response => {
                console.log(response.data);
                this.data = response.data.features;      
            });
           
        },
    };
</script>