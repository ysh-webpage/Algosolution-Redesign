<template>
    <v-app>
        <v-main>
            <v-container>
                <v-row>
                    <template v-for = 'time, filename in f.mc' :key = time>
                        <v-col cols = 12 md = 4 lg = 3 v-if = 'filename in f.tags'>
                            <v-dialog v-model = f.opened[filename]>
                                <template #activator = '{props: p}'>
                                    <v-hover v-bind = p>
                                        <template #default = '{isHovering, props}'>
                                            <v-card v-bind = props :color = 'isHovering ? `primary` : undefined' :title = filename :subtitle = time @click = 'goto(filename)' v-ripple>
                                                <v-divider />
                                                <v-chip v-for = 'tag in f.tags[filename]' :key = tag> {{ tag }} </v-chip>
                                            </v-card>
                                        </template>
                                    </v-hover>
                                </template>

                                <template #default = '{isActive}'>
                                    <template v-if = isActive>
                                        <v-card :title = filename>
                                            <template v-if = 'filename in f.urls'>
                                                <a :href = 'f.urls[filename][0]'>
                                                    <v-card variant = tonal title = Code />
                                                </a>
                                                <a :href = 'f.urls[filename][1]'>
                                                    <v-card variant = tonal title = Problem />
                                                </a>
                                            </template>
                                        </v-card>
                                    </template>
                                </template>
                            </v-dialog>
                        </v-col>
                    </template>
                </v-row>
            </v-container>
        </v-main>
    </v-app>
</template>

<script setup>
import $ from 'jquery'
import M from 'materialize-css'
import {ref, onMounted} from 'vue'

// import { useDisplay } from 'vuetify'
// import { animate, stagger, onScroll, text } from 'animejs';

const test = ref(0);
const urls = {
    mc: 'https://raw.githubusercontent.com/mysh212/Coding/refs/heads/master/mc.info',
    tags: 'https://raw.githubusercontent.com/mysh212/Coding/refs/heads/master/tags.info'
}
const tmp = ref({
    mc: '',
    tags: ''
})
const f = ref({
    mc: {},
    tags: {},
    urls: {},
    opened: {}
})

const ok = () => {
    if(tmp.value.mc == '' || tmp.value.tags == '') return;

    var split_mc = (x) => {
        var t = -1;
        for(var i = 0, len = x.length;i<len;i++) if(x[i] == ' ') t = i;
        return [x.substr(0, t), x.substr(t + 1)];
    }

    tmp.value.mc = tmp.value.mc.map((x) => split_mc(x[0]));
    tmp.value.mc.forEach((x) => {
        f.value.mc[x[1]] = x[0];
        f.value.opened[x[1]] = false;
    });
    tmp.value.tags.forEach((x) => {
        if(x[0] in f.value.tags) f.value.tags[x[0]].push(x[1]);
        else f.value.tags[x[0]] = [x[1]];
    });
}

const goto = (x) => {
    f.value.opened[x] = true;
    if(x in f.value.urls) return;

    const get_url = (code) => {
        var now = code.split('\n', 3);
        if(now.length < 3) return null;
        now = now[2];
        
       var t = /^(?:#|\/\/)(.+)$/g;
       if(!t.test(x)) return null;

       return t.exec(x)[1];
    }

    const left = `https://github.com/mysh212/Coding/blob/master/${x}`;
    const url = `https://raw.githubusercontent.com/mysh212/Coding/refs/heads/master/${x}`;
    $.get(url, (response) => {
        f.value.urls[x] = [left, get_url(response)];
    })
}

onMounted(() => {
    M.AutoInit();
    test.value = Math.random();

    for(const i in urls) $.get(urls[i], (response) => {
        tmp.value[i] = response.trim().split('\n').map((x) => x.split('\t'));
        ok();
    })
})
</script>

<style>
* {
  transition: 1s all;
}
</style>