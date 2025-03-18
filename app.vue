<template>
    <div class="container">
        <div class="app-content">
            <p class="title">Han Name Project</p>
            <div class="fetch">
                <div class="input-area">
                    <input v-model="fetchName" type="text" placeholder="Type a name" class="name-input" @keyup.enter="fetchData(fetchName)"/>
                    <button class="input-button" @click="fetchData(fetchName)">Search</button>
                </div>

                <section class="gender-area">
                    <p v-if="fetchRespone.name">
                        {{ " From " + fetchRespone.count + " people" + fetchRespone.name +
                    " name is " + fetchRespone.probability * 100 + "%"}} <span>{{ fetchRespone.gender }}</span>
                    </p>
                </section>

                <section class="age-area">
                    <p v-if="fetchRespone.age">
                        The average age of people with this name is: <span> {{ fetchRespone.age }} </span>
                    </p>
                </section>

               <section class="nationality-area">

                   <p v-if="fetchRespone.country">
                       This name is most common in the following countries:
                   </p>
                   <img
                       v-if="fetchRespone.country"
                       :src="'https://flagsapi.com/' + fetchRespone.country.country_id + '/shiny/64.png'"
                       alt="Flag"
                   >
                   <p class="country-name" v-if="fetchRespone.country"> {{ fetchRespone.country.fullName }} ({{ (fetchRespone.country.probability * 100).toFixed(2) }}%)</p>


                   <p v-else style="color: dimgray">Type a name</p>
               </section>




            </div>
        </div>

    </div>
</template>

<script setup>
import { ref } from 'vue'

const genderApiURL = "https://api.genderize.io/?name="
const nationApiURL = "https://api.nationalize.io/?name="
const ageApiURL = "https://api.agify.io/?name="
const countryApiURL = "https://restcountries.com/v3.1/alpha/"

let fetchName = ref('')
let fetchRespone = ref({})

const fetchData = async (name) => {
    if (!name) return

    try {
        const [genderRes, nationRes, ageRes] = await Promise.all([
            fetch(genderApiURL + name).then(res => res.json()),
            fetch(nationApiURL + name).then(res => res.json()),
            fetch(ageApiURL + name).then(res => res.json())
        ])

        fetchRespone.value = { ...genderRes, age: ageRes.age } // age

        if (fetchRespone.value.gender === "male") {
            fetchRespone.value.gender = 'Male'
        } else {
            fetchRespone.value.gender = 'Female'
        }

        if (nationRes.country.length > 0) {
            let country = nationRes.country[0] // En yüksek olasılıklı ülkeyi al

            // Ülke tam adını almak için API'den çağrı yap
            const countryRes = await fetch(countryApiURL + country.country_id)
            const countryData = await countryRes.json()

            country.fullName = countryData[0].name.common // API'den gelen tam ülke adını ekle
            fetchRespone.value.country = country
        }
    } catch (error) {
        console.error("API hatası:", error)
    }
}
</script>

<style>
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    .app-content {
        width: 100%;
        .title {
            text-align: center;
            font-family: "SF Pro Display", sans-serif;
            font-size: 4rem;
            font-weight: bold;
        }
        .fetch {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 20px;
            font-family: "SF Pro Display", sans-serif;
            .input-area {
                width: 100%;
                display: flex;
                flex-direction: row;
                column-gap: 10px;
                justify-content: center;
                align-items: center;
                .name-input {
                    width: 350px;
                    height: 40px;
                    border-radius: 15px;
                    border: 1px solid dimgray;
                    padding-left: 10px;
                    font-size: 18px;
                }
                .input-button {
                    width: 100px;
                    height: 40px;
                    border-radius: 15px;
                    border: 1px solid dimgray;
                    background-color: greenyellow;
                    font-size: 16px;
                    font-weight: 600;
                }
                .input-button:hover {
                    cursor: pointer;
                }
                .name-input:focus-within {
                    outline: 1px solid mediumvioletred;
                }

            }
            .gender-area {
                span {
                    color: maroon;
                    font-weight: 700;
                }
            }
            .age-area {
                span {
                    color: darkgoldenrod;
                    font-weight: 700;
                }
            }
            .nationality-area {
                display: flex;
                flex-direction: column;
                justify-content: center;
                align-items: center;
                .country-name {
                    font-weight: 700;
                }
            }


        }
    }

}
</style>
