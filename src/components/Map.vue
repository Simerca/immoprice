<template>
<div>
    <div class="overlay" v-if="loading">
        <div class="d-flex justify-content-center">
            <div class="spinner-border m-5 text-warning" role="status">
                <span class="sr-only">Loading...</span>
            </div>
        </div>
    </div>
    <div class="col-lg-4 col-md-6 col-xs-12 text-center bg-warning fixed-top p-3">
        <div id="params" class="row">
            <div class="col-12">
                Connaitre la distance depuis un point
            </div>
            <div class="form-group my-2">
                <input type="hidden" class="form-control" v-model="form.lon" id="lon" size="10" />
                <input type="hidden" class="form-control" v-model="form.lat" id="lat" size="10" />
                <button @click="selectPoint()" class="btn btn-block" :class="{'btn-primary':this.selectingPoint,'btn-outline-primary':!this.selectingPoint}">Choisir un point sur la carte</button>
            </div>
            <div class="col-12" v-if="form.lat && form.lon">
                latitude : <b>{{form.lat}}</b><br>
                longitude : <b>{{form.lon}}</b>
            </div>

            <div class="form-group" v-if="form.lat && form.lon">
                Methode de calcul :
                <select class="form-control" v-model="form.method" id="method">
                    <option value="time">Isochrones (temps)</option>
                    <option value="distance">Isodistances (distance)</option>
                </select>
            </div>
            <div class="form-group" v-if="form.method">
                <template v-if="form.method == 'time'">
                    Limite de temps

                </template>
                <template v-else>
                    Limite de distance (m)
                </template>
                <input class="form-control" type="text" v-model="form.limit" id="limit" size="10" :placeholder="form.method == 'time'?'ex: 1h':'ex 20km'" />
                <template v-if="form.method == 'time'">
                    <select v-model="form.unit" class="form-control">
                        <option value="1" selected>Heure</option>
                        <option value="2">Minutes</option>
                    </select>
                </template>
                <template v-if="form.method == 'distance'">
                    <select v-model="form.unit" class="form-control">
                        <option value="1" selected>KM</option>
                        <option value="2">M</option>
                    </select>
                </template>
            </div>

            <div class="form-group" v-if="form.limit && form.unit">
                Type de déplacement
                <select class="form-control" v-model="form.graph" id="graph">
                    <option value="Pieton">Piéton</option>
                    <option value="Voiture">Voiture</option>
                </select>
            </div>
            <div class="form-group">
                <div class="col-12 my-2">
                    <input class="btn btn-block btn-outline-light" type="button" value="Calculer" @click="isochrone()" />
                </div>
                <div class="col-12" v-if="form.lat && form.lon">
                    <input class="btn btn-block btn-outline-danger" type="button" value="Reinitialiser" @click="resetIso()" />
                </div>
            </div>
        </div>
    </div>
    <div class="col-lg-4 col-md-6 col-xs-12 text-center bg-warning fixed-bottom p-3">
        <div class="form-row row">
            <div class="form-group col-5">
                <div class="input-group mb-3">
                    <input type="number" @change="initData()" v-model="superficie" placeholder="Superficie..." class="form-control" aria-label="Recipient's username" aria-describedby="basic-addon2">
                    <span class="input-group-text" id="basic-addon2">m2</span>
                </div>
            </div>
            <div class="form-group col-5">

                <div class="input-group mb-3">

                    <input type="number" @change="initData()" v-model="budget" placeholder="Budget..." class="form-control" aria-label="Recipient's username" aria-describedby="basic-addon2">
                    <span class="input-group-text" id="basic-addon2">€</span>
                </div>
            </div>
            <div class="form-group col-2">
                <button @click="initData()" class="btn btn-outline-light btn-block">Ok</button>
            </div>
        </div>
        <div class="col-12 text-left my-2">
            <div class="row">
                <div class="col-6">
                    <div class="col-12">
                        <div class="form-check form-switch">
                            <input class="form-check-input" v-model="school" @change="initData" type="checkbox" id="flexSwitchCheckDefault">
                            <label class="form-check-label" for="flexSwitchCheckDefault">Écoles</label>
                        </div>
                    </div>
                    <div class="col-12">
                        <div class="form-check form-switch">
                            <input class="form-check-input" v-model="monumentShow" @change="initData" type="checkbox" id="flexSwitchCheckDefault">
                            <label class="form-check-label" for="flexSwitchCheckDefault">Monuments</label>
                        </div>
                    </div>
                </div>
                <div class="col-6">
                    <div class="col-12">
                        <div class="row text-left">
                            <div class="col">
                                <div style="background-color:yellow;height:10px;width:10px;display:inline-block"></div> {{prixMin}} €/m2
                            </div>
                            <div class="col">
                                <div style="background-color:red;height:10px;width:10px;display:inline-block"></div> {{prixMax}} €/m2
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-12">
            <span class="text-white" style="font-size:.5em">Développé avec ♥️ par jpb-labs</span>
        </div>
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
import ecoles from '../assets/datas/ecolesfr.json'
// import omnivore from '@mapbox/leaflet-omnivore';
import chroma from 'chroma-js'
import Gp from 'geoportal-access-lib'
export default {

    data() {
        return {
            map: null,
            school: false,
            monumentShow: false,
            markers: [],
            form: {
                lat: null,
                lon: null,
                reverse: true,
                unit: 1,
                limit: null,
                graph: null,
                method: null,
            },
            prixMax: 0,
            prixMin: 0,
            selectingPoint: false,
            url: null,
            superficie: 60,
            budget: 400000,
            loading: false,
            isoLayers: [],
        }
    },
    methods: {
        isochrone() {
            console.log(this.form);
            var lon = this.form.lon;
            var lat = this.form.lat;
            var unit = this.form.unit;
            var reverse = this.form.reverse;
            var limit = this.form.limit;
            var graph = this.form.graph;
            var method = this.form.method;
            try {
                if (method == 'distance' && unit == 1) {
                    limit = limit * 1000;
                }
                if (method == 'time' && unit == 1) {
                    limit = limit * 3600;
                }
                if (method == 'time' && unit == 2) {
                    limit = limit * 60;
                }
                console.log(limit);
                Gp.Services.isoCurve({
                    position: {
                        x: lon,
                        y: lat
                    },
                    time: (method == "time" ? limit : null),
                    distance: (method == "distance" ? limit : null),
                    graph: graph,
                    reverse: reverse,
                    apiKey: "jhyvi0fgmnuxvfv0zjzorvdn",
                    onSuccess: function (result) {
                        // resultDiv.innerHTML = "<p>" + JSON.stringify(result) + "</p>";
                        console.log(result)
                        // affichage sur la carte
                        var randomColor = '#' + Math.floor(Math.random() * 16777215).toString(16);
                        let layer = L.geoJson(result.geometry, {
                            color: randomColor
                        }).addTo(this.map);
                        this.isoLayers.push(layer);
                    }.bind(this),
                    onFailure: function (error) {
                        console.log(error)
                    }
                });
            } catch (e) {
                console.log(e);
            }
        },
        resetIso() {
            this.form = {
                lat: null,
                lon: null,
                reverse: true,
                unit: 1,
                limit: null,
                graph: null,
                method: null,
            }
            this.isoLayers.forEach(element => {
                element.remove();
            })
        },
        selectPoint() {
            this.form = {
                lat: null,
                lon: null,
                reverse: true,
                unit: 1,
                limit: null,
                graph: null,
                method: null,
            }
            this.selectingPoint = true;
        },
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
            this.map.on('click', function (e) {
                if (this.selectingPoint) {
                    console.log(e);
                    let marker = L.circleMarker(e.latlng, {
                        color: '#6FC2FF'
                    }).addTo(this.map)
                    this.isoLayers.push(marker);
                    this.form.lat = e.latlng.lat
                    this.form.lon = e.latlng.lng
                    this.selectingPoint = false;
                }
            }.bind(this))
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

                // omnivore.topojson('/ecole5.json').addTo(this.map);
                console.log(ecoles);

                ecoles.objects['ecolesfr'].geometries.filter(element => {
                    // console.log(this.map.getBounds().contains([city.latitude, city.longitude]));
                    if (this.map.getBounds().contains([element.coordinates[1], element.coordinates[0]]) && this.school) {
                        let popup = '';
                        if (element.properties.name != null) {

                            let colors = {
                                "école": "🏫",
                                "maternelle": "👶",
                                "collège": "🏫",
                                "college": "🏫",
                                "élémentaire": "🏫",
                                "primaire": "🏫",
                                "secondaire": "🏫",
                                "lycée": "🧑‍🎓",
                                "lycée professionelle": "🧑‍🎓",
                                "IUT": "🧑‍🎓"
                            }
                            if (element.properties['school-FR'] == null) {
                                element.properties['school-FR'] = 'école';
                            } else {
                                popup += `<b class="text-uppercase">${element.properties['school-FR']}</b><br>`;
                            }

                            popup += `<span>${element.properties.name}</span>`;
                            const size = 18;
                            const iconOptions = {
                                iconSize: [size, size],
                                iconAnchor: [size / 2, size + 9],
                                className: 'marker-emojis',
                                //runner, medium skin tone, Zero-Width-Joiner, female:
                                html: colors[element.properties['school-FR']] // or: '&#x1f3c3;&#x1f3fd;&#x200d;&#x2640;'
                            }

                            const markerOptions = {
                                icon: L.divIcon(iconOptions)
                            }
                            if (colors[element.properties['school-FR']]) {
                                let marker = L.marker([element.coordinates[1], element.coordinates[0]], markerOptions).bindPopup(popup)
                                    .addTo(this.map)
                                this.markers.push(marker);
                            }
                        }
                    }
                })
                monuments.filter(element => {
                    // console.log(this.map.getBounds().contains([city.latitude, city.longitude]));
                    if (this.map.getBounds().contains([element.latitude, element.longitude]) && this.monumentShow) {
                        let popup = `
                            ${element.name}<br>
                            <a class="btn btn-primary text-white font-weight-bold btn-sm" target="_blank" href="https://fr.wikipedia.org/wiki/${element.name}">Wikipedia</a>
                          `;
                        let marker = L.circleMarker([element.latitude, element.longitude], {
                            radius: 20,
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
                                    radius: 800,
                                    fillOpacity: .7,
                                    fillColor: colors[element.pmv - 1],
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

.marker-emojis {
    font-size: 2em;
}
</style>
