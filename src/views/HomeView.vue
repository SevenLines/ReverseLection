<script setup>
import { computed, ref, watch, toRaw } from "vue"
import _ from 'lodash'
import RNG from '@/utilities'
import QRCode from 'qrcode'

let count = ref(parseInt(localStorage.count || "35"));
let groupsCount = ref(4);
let subGroupsCount = ref(2);
let lateCount = ref(parseInt(localStorage.lateCount || "2"));
let seed = ref(parseInt(localStorage.seed || "0"));
let docladNumber = ref(1);

let urls = ref([
    "https://doka.guide/tools/how-the-browser-creates-pages/",
    "https://habr.com/ru/companies/skillfactory/articles/523130/",
    "https://skillbox.ru/media/code/mediazaprosy-v-css-kak-nastroit-adaptivnuyu-vyerstku-sayta/",
    "https://tproger.ru/translations/complete-sass-guide",
])
let qrCodes = ref([]);



watch(seed, () => {
    localStorage.seed = toRaw(seed.value)
})


watch(count, () => {
    localStorage.count = toRaw(count.value)
})

watch(lateCount, () => {
    localStorage.lateCount = toRaw(lateCount.value)
})


watch(urls, async () => {
    let data = [];
    await Promise.all(urls.value.map(async (x) => {
        data.push(await QRCode.toDataURL(x))
    }))
    qrCodes.value = data;
}, {
    immediate: true
})


const numbers = computed(() => {
    const rng = new RNG(seed.value)

    let result = _(Array(parseInt(count.value)).fill())
        .map((element, index) => index + 1)
        .orderBy(x => rng.nextInt())
        .concat(Array(parseInt(lateCount.value)).fill().map((x, index) => count.value + index + 1))
        .value()
    return result
})


const globalGroups = computed(() => {
    let result = {}
    numbers.value.map((x, index) => {
        if (!result[index % groupsCount.value])
            result[index % groupsCount.value] = []
        result[index % groupsCount.value].push(x)
    })
    return result;
})


const groups = computed(() => {
    let result = _.cloneDeep(globalGroups.value);

    for (let key in result) {
        let group_numbers = result[key];
        let subgroup_numbers = {};
        for (let i = 0; i < group_numbers.length; i++) {
            let el = group_numbers[i];
            let group_number = i % subGroupsCount.value;
            subgroup_numbers[group_number] = subgroup_numbers[group_number] || [];
            subgroup_numbers[group_number].push(el);
        }
        result[key] = subgroup_numbers;
    }

    return result
})

const pairs = computed(() => {
    let previousPair = []
    let result = []
    let withoutGroups = []
    for (let i = 0; i < groupsCount.value; i += 2) {
        for (let sg = 0; sg < subGroupsCount.value; ++sg) {
            let max = Math.max(groups.value[i][sg].length, groups.value[i + 1][sg].length)
            for (let k = 0; k < max; ++k) {
                if (groups.value[i + 1][sg][k]) {
                    previousPair = _.orderBy([groups.value[i][sg][k], groups.value[i + 1][sg][k]])
                    result.push(previousPair)
                } else {
                    withoutGroups.push({
                        pairCount: result.length,
                        id: groups.value[i][sg][k]
                    })
                }
            }
        }
    }
    if (withoutGroups.length) {
        result[parseInt(withoutGroups[0].pairCount) % (result.length)] = _.orderBy([...result[parseInt(withoutGroups[0].pairCount) % (result.length)], withoutGroups[0].id])
    }

    return result
})

const pairsSorted = computed(() => {
    return _.sortBy(pairs.value, x => x[0])
})

const doclads = computed(() => {
    const rng = new RNG(seed.value)
    let result = []
    for (let k = 0; k < groupsCount.value; ++k) {
        let questions = []
        for (let i = 0; i < groupsCount.value; ++i) {
            if (i != k) {
                questions.push(`${i + 1}.1`)
                questions.push(`${i + 1}.2`)
            }
        }
        let i1 = (k % 2)
        let i2 = (i1 + 1) % 2
        result.push({
            "выступает": `${k + 1}.${i1 + 1}`,
            "помогает": `${k + 1}.${i2 + 1}`,
            "готовят вопросы": questions
        })
    }
    return result
})

</script>

<template>
    <div class="container position-relative">


        <div class="row align-items-center">
            <div class="col">
                <div class="form-floating mt-2">
                    <input type="number" class="form-control" id="floatingInput" v-model="count">
                    <label for="floatingInput">Народу</label>
                </div>
            </div>
            <div class="col">
                <div class="form-floating mt-2">
                    <input type="number" class="form-control" id="floatingInput" v-model="lateCount">
                    <label for="floatingInput">Опоздуны</label>
                </div>
            </div>
            <div class="col">
                <div class="form-floating mt-2">
                    <input type="number" class="form-control" id="floatingInput" v-model="seed">
                    <label for="floatingInput">Зернышко</label>
                </div>
            </div>
            <!-- <div class="col">
                <div class="timer">
                    05:00
                </div>
            </div> -->
        </div>

        <ul class="nav nav-tabs mt-2" id="myTab" role="tablist">
            <li class="nav-item" role="presentation">
                <button class="nav-link active" id="home-tab" data-bs-toggle="tab" data-bs-target="#home-tab-pane"
                    type="button" role="tab" aria-controls="home-tab-pane" aria-selected="true">№1 Читаем статью</button>
            </li>
            <li class="nav-item" role="presentation">
                <button class="nav-link" id="profile-tab" data-bs-toggle="tab" data-bs-target="#profile-tab-pane"
                    type="button" role="tab" aria-controls="profile-tab-pane" aria-selected="false">№2 Рассказываем друг
                    дружке</button>
            </li>
            <li class="nav-item" role="presentation">
                <button class="nav-link" id="contact-tab" data-bs-toggle="tab" data-bs-target="#contact-tab-pane"
                    type="button" role="tab" aria-controls="contact-tab-pane" aria-selected="false">№3 Подготовка
                    доклада</button>
            </li>
            <li class="nav-item" role="presentation">
                <button class="nav-link" id="end-tab" data-bs-toggle="tab" data-bs-target="#end-tab-pane" type="button"
                    role="tab" aria-controls="end-tab-pane" aria-selected="false">№4 Защита</button>
            </li>
        </ul>

        <div class="tab-content" id="myTabContent">
            <div class="tab-pane fade show active" id="home-tab-pane" role="tabpanel" aria-labelledby="home-tab"
                tabindex="0">

                <ul class="instructions">
                    <li>прочитайте статью</li>
                    <li>выделите 5 основных тезисов</li>
                    <li>приготовьтесь поделиться тезисами и полученными знаниями о технологии</li>
                </ul>
                <div class="d-grid" style=" grid-template-columns: repeat(2, 1fr); gap: 8px">
                    <div v-for="(g, k) in globalGroups"
                        class="border rounded p-3 mb-2 position-relative overflow-hidden article-command">
                        <h3>Статья {{ parseInt(k) + 1 }}</h3>

                        <span class="highlighted" style="font-size: 2rem;">{{ _.orderBy(g).join(", ") }}</span>
                        <img class="qr-code" :src="qrCodes[k]" alt="">
                    </div>
                </div>
            </div>
            <div class="tab-pane fade" id="profile-tab-pane" role="tabpanel" aria-labelledby="profile-tab" tabindex="0">
                <ul class="instructions">
                    <li>расскажите соседу об изученой технологии</li>
                    <li>задайте соседу минимум один вопрос</li>
                </ul>
                <div class="display-6 text-center" style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px">
                    <div v-for="p in pairsSorted">
                        <span class="highlighted" v-for="(i, index) in p">
                            <b>{{ i }} </b>
                            <span v-if="index < p.length - 1">&nbsp;vs&nbsp;
                            </span>
                        </span>
                    </div>
                </div>
            </div>
            <div class="tab-pane fade" id="contact-tab-pane" role="tabpanel" aria-labelledby="contact-tab" tabindex="0">
                <ul class="instructions">
                    <li>объединитесь в команды</li>
                    <li>сформулируйте 10 тезисов по статье</li>
                    <li>приготовьтесь отправить докладчика выступить у доски</li>
                </ul>
                <div class="p-2" style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px">

                    <div v-for="(g, k) in groups">
                        <div v-for="(numbers, index) in g" class="border rounded p-3 mb-2">
                            <h3>Команда {{ parseInt(k) + 1 }}.{{ parseInt(index) + 1 }}</h3>
                            <span class="highlighted" style="font-size: 1.5rem;">{{ _.orderBy(numbers).join(", ")
                            }}</span>
                        </div>
                    </div>
                </div>
            </div>
            <div class="tab-pane fade" id="end-tab-pane" role="tabpanel" aria-labelledby="end-tab" tabindex="0">
                <nav aria-label="Page navigation example">
                    <ul class="pagination mt-2">
                        <template v-for="p in groupsCount">
                            <li class="page-item" :class="{ active: p == docladNumber }" @click="docladNumber = p">
                                <a class="page-link" href="#">{{ p }}</a>
                            </li>
                        </template>
                    </ul>
                </nav>
                <h1>Выступает команда: <span class="highlighted">{{ doclads[docladNumber - 1]['выступает'] }}</span></h1>
                <h1>Помогает команда: <span class="highlighted">{{ doclads[docladNumber - 1]['помогает'] }}</span></h1>
                <h1>Готовят вопросы:
                    <span class="highlighted">{{ doclads[docladNumber - 1]['готовят вопросы'].join(", ") }}</span>
                </h1>   
            </div>
        </div>
    </div>
</template>

<style lang="scss">
.highlighted {
    background-color: yellow;
    white-space: nowrap;
}

.instructions {
    font-size: 1.5rem;
}

.timer {
    cursor: pointer;
    z-index: 1000;
    background-color: white;
    font-size: 4rem;
    color: red;
}


.article-command:hover {
    cursor: pointer;
    .qr-code {
        right: 0;
    }
}

.qr-code {
    position: absolute;
    top: 0;
    bottom: 0;
    height: 100%;
    right: -50%;
    transition: all .3s;
}
</style>