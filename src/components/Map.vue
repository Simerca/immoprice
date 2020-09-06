<template>
<div>
    <div class="overlay" v-if="loading">
        <div class="d-flex justify-content-center">
            <div class="spinner-border m-5 text-warning" role="status">
                <span class="sr-only">Loading...</span>
            </div>
        </div>
    </div>
    <div class="col-md-12 fixed-bottom">
        <button class="btn btn-primary btn-block" @click="prepareData()">Check !</button>
    </div>
    <div class="map" ref="map"></div>
</div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import datas from '../assets/datas/pvm.json'
import villes from '../assets/datas/ville_france.json'
import chroma from 'chroma-js'
export default {

    data() {
        return {
            map: null,
            markers: [],
            loading: false,
        }
    },
    methods: {
        initMap() {
            this.map = L.map(this.$refs.map, {
                renderer: L.canvas(),
                minZoom:11
            });
            this.map.setView([44.8333, -0.5667], 13);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
            }).addTo(this.map);
        },
        prepareData() {
            this.loading = true;
            setTimeout(function(){
                this.initData().then(() => {
                    this.loading = false;
                })
            }.bind(this),100)
        },
        async initData() {

            return await new Promise((resolve) => {
                this.markers.forEach(marker => {
                    marker.remove();
                })
                this.markers = [];
                let prixmin = 0;
                let prixmax = 0;
                villes.forEach(element => {
                    let pvm = datas.find(map => map.INSEE_COM == element.codeinsee);
                    if (pvm) {
                        if (this.map.getBounds().contains([element.latitude, element.longitude])) {
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
                let colors = chroma.scale(['yellow', 'red']).domain([prixmin, prixmax]).colors(prixmax);
                villes.forEach(element => {
                    let pvm = datas.find(map => map.INSEE_COM == element.codeinsee);
                    if (pvm) {
                        if (this.map.getBounds().contains([element.latitude, element.longitude])) {
                            element.pmv = pvm.PrixMoyen_M2
                            if (colors[element.pmv - 1]) {
                                let popup = L.popup()
                                    .setContent(element.nom_commune + '<br>PrixM2 : ' + element.pmv + '€');
                                let marker = L.circleMarker([element.latitude, element.longitude], {
                                    color: colors[element.pmv - 1]
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
