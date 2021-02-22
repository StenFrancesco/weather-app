<template>
    <div class="text-white mb-8">
        
        <div class="places-input text-gray-800">
            <input type="search" id="address" class="form-control rounded" placeholder="Where are we going?" />
            
            <p class="m-1">Selected: <strong id="address-value">none</strong></p>
        </div>

        <div class="weather-container font-sans w-128 max-w-lg overflow-hidden bg-gray-500 shadow-lg mt-4 rounded">
            <div class="current-weather flex items-center justify-between px-6 py-8">
                <div class="flex items-center">
                    <div>
                        <div class="text-6xl font-semibold">{{ currentTemperature.actual }}° C</div>
                        <div>Feels like: {{ currentTemperature.feels }}° C</div>
                    </div>
                    <div class="mx-5">
                        <div class="font-semibold">{{ currentTemperature.summary }}</div>
                        <div>{{ location.city }}</div>
                    </div>
                </div>
                <div>
                    <canvas ref="iconCurrent" id="iconCurrent" width="96" height="96"></canvas>    
                </div>                
            </div> 
            <div class="future-weather text-sm bg-gradient-to-tr from-blue-200 to-blue-400 px-6 py-8 overflow-hidden">
                <div
                 v-for="(days, index) in daily" 
                 :key="days.datetime" 
                 class="flex items-center"
                 :class="{ 'mt-8' : index > 0 }"
                 v-if="index < 5"
                 >
                    <div class="w-1/6 text-lg text-gray-200">{{ toDayOfWeek(days.datetimeEpoch) }}</div>
                    <div class="w-4/6 px-5 flex items-center">
                        <div>
                            <canvas :id="`icon${index+1}`" :data-icon="toKebabCase(days.icon)" width="24" height="24"></canvas>
                        </div>
                        <div class="ml-3">{{ days.conditions }}</div>
                    </div>
                    <div class="w-1/6 text-right">
                        <div>{{ Math.round(days.tempmax) }}° C</div>
                        <div>{{ Math.round(days.tempmin) }}° C</div>
                    </div>
                </div>                
            </div> 
        </div> 
    </div>
</template>

<script>
    export default {
        mounted() {
            this.fetchData()
            
            var placesAutocomplete = places({
                appId: 'pl6J0VA98HU5',
                apiKey: 'af866332d190134bd29827b3fd949fc6',
                container: document.querySelector('#address')
            }).configure({
                type: 'city',
                aroundLatLngViaIP: false,
            });

            var $address = document.querySelector('#address-value')
            
            placesAutocomplete.on('change', (e) => {
                $address.textContent = e.suggestion.value

                this.location.city = `${e.suggestion.name}, ${e.suggestion.country}`
                this.location.lat = `${e.suggestion.latlng.lat}`
                this.location.lng = `${e.suggestion.latlng.lng}`                
            });

            placesAutocomplete.on('clear', function () {
                $address.textContent = 'none';
            })
        },
        watch: {
            location: {
                handler(newValue, oldValue) {
                    this.fetchData()
                },
                deep: true
            }
        },
        data() {
            return {
                currentTemperature: {
                    actual: '',
                    feels: '',
                    summary: '',
                    icon: '',
                },
                daily: [],
                location: {
                    city: 'Heiloo',
                }
            }
        },
        methods: {
            fetchData(){
                var skycons = new Skycons({'color': 'white'});

                fetch(`/api/weather?city=${this.location.city}`)
                .then(response => response.json())
                .then(data => {
                  
                    this.currentTemperature.actual = Math.round(data.currentConditions.temp)
                    this.currentTemperature.feels = Math.round(data.currentConditions.feelslike)
                    this.currentTemperature.summary = data.currentConditions.conditions
                    this.currentTemperature.icon = this.toKebabCase(data.currentConditions.icon)
                  
                    this.daily = data.days

                    skycons.add("iconCurrent", this.currentTemperature.icon);
                    skycons.play();
                    
                    this.$nextTick(() => {
                        skycons.add("icon1", document.getElementById('icon1').getAttribute('data-icon'))
                        skycons.add("icon2", document.getElementById('icon2').getAttribute('data-icon'))
                        skycons.add("icon3", document.getElementById('icon3').getAttribute('data-icon'))
                        skycons.add("icon4", document.getElementById('icon4').getAttribute('data-icon'))
                        skycons.add("icon5", document.getElementById('icon5').getAttribute('data-icon'))
                        skycons.play()
                    })
                    

                })
            },
            toKebabCase(stringToConvert) {
                return stringToConvert.split(' ').join('-');
            },
            toDayOfWeek(timestamp) {
                const newDate = new Date(timestamp*1000)
                const days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']

                return days[newDate.getDay()]
            }
        }
    }
</script>
