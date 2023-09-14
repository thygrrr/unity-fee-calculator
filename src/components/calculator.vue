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
                <h3>Last 1 to 12 Months</h3>
                <li>12-Month Revenue: $<input type="number" v-model='revenue' /></li>
                <li>Lifetime Installs until now:<input type="number" v-model='installs' /></li>
                <br/>
                <h3>Next Period Projections</h3>
                <li>Revenue: $<input type="number" v-model='futureRevenue' /></li>
                <li>Installs:<input type="number" v-model='futureInstalls' /></li>
                <br/>
                <li>Publisher / App Store Cuts:<input type="number" v-model='storeCuts' /></li>
                <li><a href="https://en.wikipedia.org/wiki/Operating_expense">Operating Expenses</a>:<input type="number" v-model='opEx' /></li>
                <br/>
                <li><div></div><div style="text-align: center; width: 50%"><button v-on:click="doMath">Hit Me</button></div></li>
                <br/>
                <li class="amount" v-bind:class="youOwe > 0 ? 'bad': 'good'">You Owe: {{formatMoola()}}</li>
                <li class="amount" v-bind:class="youOwe > 0 ? 'bad': 'good'">Unity's Revenue Share: {{formatRevShare()}}</li>
                <li class="amount" v-bind:class="youOwe">Your Revenue per Install: {{formatRPI()}}</li>
                <br/>
                <b><li class="amount" v-bind:class="youOwe">Your <a href="https://en.wikipedia.org/wiki/Earnings_before_interest_and_taxes">EBIT</a> per Install: {{formatEarnings()}}</li></b>
            </ul>            
        </div>
        <div>
            <h1 style="color: #555;">What's this all about?</h1>
            <ul>
                <li>Unity is going to charge devs <b>per install</b>.</li>
                <li>Read more at the <a href="https://blog.unity.com/news/plan-pricing-and-packaging-updates">Unity Blog</a></li>
                <li>Join the conversation at the <a href="https://forum.unity.com/threads/unity-plan-pricing-and-packaging-updates.1482750">Unity Forums</a> or the Unity <a href="https://discord.gg/unity">discord</a></li>
            </ul>
            <a href="https://twitter.com/kurtruslfanclub/status/1701621432894267593"><img src="tweet.png" /></a>
        </div>
    </div>
    <div class="footer">Made by <a href="https://www.threads.net/@skidvis">Skid Vis</a> from <a href="https://youtube.com/sharkjets">SharkJets</a> - <a href="https://github.com/skidvis/unity-fee-calculator">Fork</a> around and find out. </div>
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
                installs: 200000,
                futureInstalls: 200000,
                futureRevenue: 200000,
                storeCuts : 0.3,
                opEx : 100000,
                opExPerInstall : 0,
                youOwe : 0,
                revenuePerInstall : 0,
                revenueShare : 0,
                subscriptions: [
                    {key: 0, value: 'Personal/Plus'}, 
                    {key: 1, value: 'Pro'}, 
                    {key: 2, value: 'Enterprise'}
                ],
                thresholdsRevenue: [200_000, 1_000_000, 1_000_000],
                thresholdsInstalls: [200_000,1_000_000,1_000_000],
                surplusBrackets: [100_000,400_000,1_000_000],
                rates: [
                    {costs: [0.2,0.2,0.2,0.2]},
                    {costs: [0.15,0.075,0.03,0.02]},
                    {costs: [0.125,0.06,0.02,0.01]}
                ]
            }
        },
        mounted() {
            this.doMath();
        },
        methods: {
            doMath(){
                //1-100k, 100001-500k, 500001-1M, 1000001+
                let surplusInstalls = this.futureInstalls + this.installs - this.thresholdsInstalls[this.inputSubscription];    

                console.log(`installs: ${this.installs}, surplusInstalls: ${surplusInstalls}`);
                this.youOwe = 0;
                this.revenueShare = 0;
                this.opExPerInstall = this.opEx / this.futureInstalls;
                this.revenuePerInstall = this.revenue / this.installs;
                if(this.remainingInstalls <= this.thresholdsInstalls[this.inputSubscription] || this.revenue < this.thresholdsRevenue[this.inputSubscription]) return;

                console.log(`${surplusInstalls} <= ${this.thresholdsInstalls[this.inputSubscription]} && ${this.revenue} <= ${this.thresholdsRevenue[this.inputSubscription]}`)
                
                //We deduct each bracket individually, clamping between 0 and the bracket size
                var costs = this.rates[this.inputSubscription].costs;
                for(var i = 0; i < this.thresholdsRevenue.length; i++)
                {
                    var surplus = Math.min(surplusInstalls, this.surplusBrackets[i]);
                    this.youOwe += Math.max(0, surplus * costs[i]);
                    surplusInstalls -= surplus;                    
                }
                this.youOwe += Math.max(0, surplusInstalls * costs[costs.length-1]);
                this.revenuePerInstall = this.futureRevenue / this.futureInstalls;
                this.revenueShare = this.youOwe / this.futureRevenue;
            },
            formatMoola(){
                return this.youOwe.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
            },
            formatRPI(){
                return this.revenuePerInstall.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
            },
            formatRevShare(){
                const revenueSharePct = this.revenueShare * 100;
                return revenueSharePct.toFixed(2) + "%";
            },
            formatEarnings(){
                return (this.revenuePerInstall - this.revenuePerInstall * (this.revenueShare+this.storeCuts) - this.opExPerInstall).toLocaleString('en-US', { style: 'currency', currency: 'USD' });
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
