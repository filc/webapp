<template>
    <div class="screen">
        <div class="head">
            <div class="searchbox"><input type="text" placeholder="Házik keresése" v-model="query"></div>
            <button class="search">
                <Icon src="fi/search" />
            </button>
        </div>

        <HomeworkList :list="getTomorrow()" title="Holnapra" />
        <HomeworkList :list="getActual()" title="Aktuális" />
        <HomeworkList :list="getExpired()" title="Lejárt határidejű" />
    </div>
</template>

<script>
import GlobalState from '../globalState'

import HomeworkList from '../components/HomeworkList.vue';
import { getHWCompObjFArr } from '../dataHandler';
import { getDateCompareNumber, formatURLsHTML, htmlToText } from '../util';
import { isHomeworkDone } from '../api';


export default {
    name:"Homework",
    data(){
        return {
            query:"",
            GlobalState,
            homeworks:GlobalState.processedData.homeworks,
            homeworksCompleted:GlobalState.processedData.homeworksCompleted,
        }
    },
    methods:{
        isCompleted(hw){
            return hw.IsMegoldva;
            /* return getHWCompObjFArr(hw.Id, this.homeworksCompleted)?.value == true; */
        },
        sortByDone(a,b){
            return this.isCompleted(a.homework) - this.isCompleted(b.homework);
        },
        isExpired(e){
            let g = getDateCompareNumber;
            return g(new Date(e.homework.Hatarido)) < g(new Date());
        },
        isTomorrow(e){
            if (!GlobalState.options["homeworks.separateTomorrow"]){
                return false;
            }
            let g = getDateCompareNumber;
            let tomorrow = new Date();
            tomorrow.setDate(tomorrow.getDate()+1);
            let compare = g(new Date(e.homework.Hatarido));
            return compare == g(new Date()) || compare == g(tomorrow);
        },
        getActual(){
            return this.getFiltered().filter((e)=>{
                return !this.isExpired(e) && !this.isTomorrow(e);
            }).sort((a,b)=>{
                let g = x => new Date(x.homework.Hatarido);
                return g(a)-g(b);
            }).sort(this.sortByDone)
        },
        getTomorrow(){
            return this.getFiltered().filter((e)=>{
                return this.isTomorrow(e);
            }).sort((a,b)=>{
                let g = x => new Date(x.homework.Hatarido);
                return g(a)-g(b);
            }).sort(this.sortByDone)
        },
        getExpired(){
            
            return this.getFiltered().filter((e)=>{
                return this.isExpired(e);
            }).sort((a,b)=>{
                let g = x => new Date(x.homework.Hatarido);
                return g(b)-g(a);
            }).sort(this.sortByDone)
        },
        getFiltered(){
            let keywords = this.query.split(" ").filter(e=>e);
            
            if (keywords.length == 0){
                keywords.push("");
            }
            
            
            function norm(s){
                return s.toLowerCase();
            }

            return this.homeworks.filter(hw=>{
                let map = new Map();
                keywords.forEach(e => {
                    map.set(e,0);
                });
                for (let key of keywords){
                    let check = [
                        hw.homework.Tantargy,
                        hw.homework.Rogzito,
                        htmlToText(formatURLsHTML(hw.homework.Szoveg))
                    ]
                    for (let c of check){
                        if (norm(c).indexOf(norm(key)) != -1){
                            map.set(key,map.get(key)+1);                        
                        }
                    }
                    
                    
                }
                
                for (const e of map.values()) {
                    if (e == 0){
                        return false;
                    }
                }
                return true;
            });
        },
    },
    components:{
        HomeworkList
    }
}
</script>

<style scoped>
    .head {
        padding: 20px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: row;
    }
    .head .searchbox {
        flex: 1;
        margin-right: 20px;
        overflow: hidden;
    }
    .head .search, .head .searchbox {
        border-radius: 22px;
        height: 44px;
        box-sizing: border-box;
        background-color: var(--element-color);
        box-shadow: var(--elem-shadow);
    }
    .head .searchbox input {
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        border: none;
        outline: none;
        font-size: 18px;
        padding: 5px 20px;
        background: none;
    }
    .head .search {
        
        flex:none;
        width: 44px;
        
        padding: 0;
        display: flex;
        border: none;
        justify-content: center;
        align-items: center;
        
        outline: none;
    }
    /deep/ .head .search .icon svg {
        width: 20px;
        height: 20px;
        opacity: 0.65;
    }
</style>