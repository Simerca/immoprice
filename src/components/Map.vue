<template>
<div>
    <div class="overlay" v-if="loading">
        <div class="d-flex justify-content-center">
            <div class="spinner-border m-5 text-warning" role="status">
                <span class="sr-only">Loading...</span>
            </div>
        </div>
    </div>
    <div class="col-md-12 text-center bg-warning fixed-bottom p-3">
        <div class="form-group">
            Superficie en m2 habitable
            <input type="number" @change="initData()" v-model="superficie" placeholder="Superficie..." class="form-control" />
        </div>
        <div class="form-group">
            Votre Budget en euros
            <input type="number" @change="initData()" v-model="budget" placeholder="Budget..." class="form-control" />
        </div>
        <div class="form-group">
            <button @click="initData()" class="btn btn-outline-light btn-block">Rechercher</button>
        </div>
        <div class="col-12">
            <div class="row">
                <div class="col-6">
                    <div style="background-color:yellow;height:10px;width:10px;display:inline-block"></div> {{prixMin}} €/m2
                </div>
                <div class="col-6">
                    <div style="background-color:red;height:10px;width:10px;display:inline-block"></div> {{prixMax}} €/m2
                </div>
            </div>
        </div>
        <span class="text-white">Développé avec ♥️ par <a href="https://previmeteo.com"><img style="height:16px;" src="https://previmeteo.com/services-meteo-pro/wp-content/uploads/2019/01/logo-previmeteo.png" class="img-fluid"></a></span>
    </div>
    <div class="map" ref="map"></div>
</div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import datas from '../assets/datas/pvm.json'
import villes from '../assets/datas/ville_france.json'
import monuments from '../assets/datas/monuments.json'
import chroma from 'chroma-js'
export default {

    data() {
        return {
            map: null,
            markers: [],
            prixMax: 0,
            prixMin: 0,
            superficie: 60,
            budget: 400000,
            loading: false,
        }
    },
    methods: {
        initMap() {
            this.map = L.map(this.$refs.map, {
                renderer: L.canvas(),
                minZoom: 9
            });
            this.map.setView([44.8333, -0.5667], 13);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);
            this.prepareData()
            this.map.on('moveend', function () {
                this.initData()
            }.bind(this));
        },
        prepareData() {
            this.loading = true;
            setTimeout(function () {
                this.initData().then(() => {
                    this.loading = false;
                })
            }.bind(this), 100)
        },
        async initData() {

            return await new Promise((resolve) => {
                this.markers.forEach(marker => {
                    marker.remove();
                })
                this.markers = [];
                let prixmin = 999999;
                let prixmax = 0;

                let villesFiltered = villes.filter(city => {
                    // console.log(this.map.getBounds().contains([city.latitude, city.longitude]));
                    if (this.map.getBounds().contains([city.gps_lat, city.gps_lng])) {
                        let pvm = datas.find(map => map.INSEE_COM == city.insee_code);
                        if (pvm) {
                            let prixm = pvm.PrixMoyen_M2;
                            if ((prixm * this.superficie) <= this.budget) {
                                return true;
                            }
                        }
                    }
                })
                monuments.filter(element => {
                    // console.log(this.map.getBounds().contains([city.latitude, city.longitude]));
                    if (this.map.getBounds().contains([element.latitude, element.longitude])) {
                          let popup = `
                            ${element.name}
                          `;
                          let marker = L.circleMarker([element.latitude, element.longitude], {
                                    radius:20,
                                    color: 'blue',
                                }).bindPopup(popup).addTo(this.map)
                                this.markers.push(marker);
                    }
                })
                console.log('ville filtré');
                console.log(villesFiltered.length);
                villesFiltered.forEach(element => {
                    let pvm = datas.find(map => map.INSEE_COM == element.insee_code);
                    if (pvm) {
                        if (this.map.getBounds().contains([element.gps_lat, element.gps_lng])) {
                            element.pmv = pvm.PrixMoyen_M2
                            if (parseInt(element.pmv) > prixmax) {
                                prixmax = element.pmv;
                            }
                            if (parseInt(element.pmv) < prixmin) {
                                prixmin = element.pmv;
                            }
                        }
                    }
                })
                console.log('prixmin');
                console.log(prixmin)
                this.prixMin = prixmin;
                console.log('prixmax');
                console.log(prixmax);
                this.prixMax = prixmax;
                let colors = chroma.scale(['yellow', 'red']).domain([prixmin, prixmax]).colors(prixmax);
                villesFiltered.forEach(element => {
                    let pvm = datas.find(map => map.INSEE_COM == element.insee_code);
                    if (pvm) {
                        if (this.map.getBounds().contains([element.gps_lat, element.gps_lng])) {
                            element.pmv = pvm.PrixMoyen_M2;

                            if (colors[element.pmv - 1]) {
                                let popuptext = `
                                    ${element.name}<br>
                                    Prix au m2 : ${element.pmv} €<br>
                                    Habitants : ${pvm.POPULATION}<br>
                                    <a class="btn btn-sm btn-warning" target="_blank" href="https://www.leboncoin.fr/recherche/?category=9&locations=${element.name}_${element.zip_code}&real_estate_type=1,2&price=min-${this.budget}">Leboncoin</a>
                                    `
                                let popup = L.popup()
                                    .setContent(popuptext);
                                let marker = L.circle([element.gps_lat, element.gps_lng], {
                                    radius:800,
                                    fillOpacity:1,
                                    fillColor:colors[element.pmv - 1],
                                    color: colors[element.pmv - 1],
                                }).bindPopup(popup).addTo(this.map)
                                this.markers.push(marker);
                            }
                        }
                    }
                })
                console.log(this.markers.length + ' charged')
                resolve()
            });
        }

    },
    mounted() {
        this.initMap();
    }

}
</script>

<style>
.map {
    height: 100vh;
}

.overlay {
    position: absolute;
    z-index: 100000;
    background-color: rgba(0, 0, 0, .6);
    height: 100%;
    width: 100vw;
}
</style>
