<template>
    <div class="container h-100">
        <div class="row h-100" v-if="!this.isLoggedIn">
            <div class="col-lg-7 my-auto">
                <div class="header-content mx-auto text-white">
                    <h1 class="mb-2">Pink Fox Datawallet</h1>
                    <h2 class="mb-5">Demoapp zur Anbindung der Datawallet SDK um Zugriff auf Facebook-Kontodaten zu erhalten.</h2>
                    <a class="btn btn-outline btn-xl" v-if="!this.isLoggedIn" v-on:click="this.login">Sign in with Datawallet</a>
                </div>
            </div>
            <div class="col-lg-5 my-auto">
                <div class="device-container">
                    <div class="device-mockup iphone6_plus portrait white">
                        <div class="device">
                            <div class="screen">


                            </div>
                            <div class="button">
                                <!-- You can hook the "home button" to some JavaScript events or just remove it -->
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="row h-100" v-if="this.isLoggedIn">
            <div class="col-12 my-auto">
                <h2 class="mb-2 text-white">Welcome, {{ this.name }}!</h2>
                <p class="text-white">
                    You have written <b>{{ this.messages.length }}</b> Messages in Facebook!
                </p>
                <div class="card">
                    <div class="card-header">Messages by Day-of-Week and time</div>
                    <div class="card-body">
                        <apexchart type=heatmap height=350  :options="chartOptions" :series="series"/>
                    </div>
                </div>
            </div>
        </div>
    </div>



</template>

<script>
    import pls from '@datawallet/pls'

    export default {
        name: 'DatawalletLogin',
        props: {},
        data: function () {
            return {
                isLoggedIn: false,
                name: '',
                messages: [],
                series: [],
                chartOptions: {
                    dataLabels: {
                        enabled: true
                    },
                    colors: ["#008FFB"],
                    plotOptions: {
                        heatmap: {
                            shadeIntensity: 0.5,
                        }
                    }
                }
            }
        },
        methods: {
            login: async function () {
                this.isLoggedIn = false;
                if (!pls.isAvailable()) {
                    // show some call to action to your user to install Datawallet 3.0
                    // and give feedback.
                    alert("You need Datawallet to access this application");
                    return;
                }

                const signedQuery = await pls.authorize(
                    {
                        query: '{ facebook { profile { first_name, last_name }, messages { content, timestamp } } }',
                        shortName: 'Pink Fox',
                        avatarUrl: 'https://via.placeholder.com/88x88.png/d01673/fff/?text=Pink%20Fox'
                    }
                );

                const data = await pls.query(signedQuery);

                this.isLoggedIn = true;
                this.name       = data.facebook.profile.first_name + ' ' + data.facebook.profile.last_name;
                this.messages   = data.facebook.messages;

                var weekday = new Array(7);
                weekday[1]  = "Monday";
                weekday[2]  = "Tuesday";
                weekday[3]  = "Wednesday";
                weekday[4]  = "Thursday";
                weekday[5]  = "Friday";
                weekday[6]  = "Saturday";
                weekday[0]  = "Sunday";

                var tmpSeriesDates = {};
                this.messages.forEach(function (message) {

                    var date      = new Date(message.timestamp * 1000);
                    var dayOfWeek = date.getUTCDay();
                    var hours     = date.getUTCHours();

                    if (typeof tmpSeriesDates[dayOfWeek] == 'undefined') {
                        tmpSeriesDates[dayOfWeek] = [];
                    }

                    if (typeof tmpSeriesDates[dayOfWeek][hours] == 'undefined') {
                        tmpSeriesDates[dayOfWeek][hours] = 0;
                    }

                    tmpSeriesDates[dayOfWeek][hours]++;
                })

                var tmpSeries = [];
                weekday.forEach(function (dayName, dayIndex) {

                    var hourlyData = [];
                    for (var hour = 0; hour < 24; hour++) {
                        var messagesData = 0;
                        if (typeof tmpSeriesDates[dayIndex][hour] != "undefined") {
                            messagesData = tmpSeriesDates[dayIndex][hour];
                        }

                        hourlyData.push({x: hour, y: messagesData})
                    }

                    tmpSeries.push(
                        {
                            name: dayName,
                            data: hourlyData
                        })
                })

                this.series = tmpSeries;

            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
