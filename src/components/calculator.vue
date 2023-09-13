<template lang="">
    <div class="content">
        <div>
            <h1>Unity Runtime Fee Calculator</h1>   
            <ul class="inputs">
                <li>
                    Selected Subscription Plan:
                    <select v-model="inputSubscription">
                        <option v-for="sub in subscriptions" v-bind:value="sub.key">{{sub.value}}</option>
                    </select>     
                </li>
                <li>12-Month Revenue: $<input type="number" v-model='revenue' /></li>
                <li>Lifetime Installs:<input type="number" v-model='installs' /></li>
                <li><div></div><div style="text-align: center; width: 50%"><button v-on:click="doMath">Hit Me</button></div></li>
                <li class="amount" v-bind:class="youOwe > 0 ? 'bad': 'good'">You Owe: {{formatMoola()}}</li>
            </ul>            
        </div>
        <div>
            <h1 style="color: #555;">What's this all about?</h1>
            <ul>
                <li>Unity is going to charge devs <b>per install</b>.</li>
                <li>Read more at the <a href="https://blog.unity.com/news/plan-pricing-and-packaging-updates">Unity Blog</a></li>
            </ul>
            <a href="https://twitter.com/kurtruslfanclub/status/1701621432894267593"><img src="/tweet.png" /></a>
        </div>
    </div>
    <div class="footer">Made by <a href="https://www.threads.net/@skidvis">Skid Vis</a> from <a href="https://youtube.com/sharkjets">SharkJets</a></div>
</template>

<script>
    

    /*
    > game makes 200k from in-app purchases after being installed 3 million times 
    > now owe Unity 20c per 2.8M installs, $560K
    > that’s 360K more than we made
    */
    

    export default {
        data() {
            return {
                inputSubscription: 0,
                revenue: 200000,
                installs: 3000000,
                youOwe : 0,
                subscriptions: [
                    {key: 0, value: 'Personal/Plus'}, 
                    {key: 1, value: 'Pro'}, 
                    {key: 2, value: 'Enterprise'}
                ],
                thresholdsRevenue: [200000, 1000000, 1000000],
                thresholdsInstalls: [200000,1000000,1000000],
                rates: [
                    {costs: [0.2,0.2,0.2,0.2]},
                    {costs: [0.15,0.075,0.03,0.2]},
                    {costs: [0.125,0.06,0.2,0.1]}
                ]
            }
        },
        mounted() {
            this.doMath();
        },
        methods: {
            doMath(){
                //1-100k, 100001-500k, 500001-1M, 1000001+
                let remainingInstalls = this.installs - this.thresholdsInstalls[this.inputSubscription];                
                console.log(`installs: ${this.installs}, remainingInstalls: ${remainingInstalls}, ${this.thresholdsInstalls[this.inputSubscription]}`);
                this.youOwe = 0;
                console.log(`${remainingInstalls} <= ${this.thresholdsInstalls[this.inputSubscription]} && ${this.revenue} <= ${this.thresholdsRevenue[this.inputSubscription]}`)
                if(this.remainingInstalls <= this.thresholdsInstalls[this.inputSubscription] || this.revenue < this.thresholdsRevenue[this.inputSubscription]) return;
                if(remainingInstalls >= 1000001) this.youOwe = remainingInstalls * this.rates[this.inputSubscription].costs[3];
                if(remainingInstalls >= 500001 && remainingInstalls <= 1000000) this.youOwe = remainingInstalls * this.rates[this.inputSubscription].costs[2];
                if(remainingInstalls >= 100001 && remainingInstalls <= 500000) this.youOwe = remainingInstalls * this.rates[this.inputSubscription].costs[1];
                if(remainingInstalls >= 1 && remainingInstalls <= 10000) this.youOwe = remainingInstalls * this.rates[this.inputSubscription].costs[0];
                this.youOwe = this.youOwe.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
            },
            formatMoola(){
                return this.youOwe.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
            }
        },
    }
</script>

<style>
    body{
        font-family: sans-serif;
        font-size: 20px;
    }

    .inputs li{
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .content{
        display: flex;
        justify-content: space-evenly;
    }

    ul{
        padding:0;
    }

    li{
        list-style-type: none;
        padding:5px;
    }

    select{
        width: 50%;
        background-color: #dddddd;
        font-size: 20px;
        text-align: center;
    }

    input, button{
        width:49%;
        font-size: 20px;
    }

    .bad{
        color: red;
    }

    .good{
        color: green;
    }

    .amount{
        width:100%;
        text-align: center;
        display: block !important;
    }

    .footer{
        text-align: center;
        font-size: 12px;
        margin-top: 50px;
        width: 100%;
    }

    @media screen and (max-width: 767px) {
        body{
            text-align: center;
        }
        .content{
            flex-direction: column;
        }

        .inputs li{
            flex-direction: column;
        }

        img{
            width:100%;
        }
    }
</style>