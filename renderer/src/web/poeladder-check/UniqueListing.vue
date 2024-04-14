<template>
  <div v-if="!error" class="layout-column min-h-0" style="height: auto;">
    <div class="mb-2 flex pl-2">
      <div class="flex items-baseline text-gray-500 mr-2">
        <span class="mr-1">{{ t(':matched') }}</span>
        <span v-if="!unique" class="text-gray-600">...</span>
        <span v-else>{{ unique.name }}</span>
      </div>
      <online-filter v-if="unique" :by-time="true" :filters="filters" />
      <div class="flex-1"></div>
    </div>
    <div class="helpText" v-if="accountNameMissing">
        <p>Add your PoE account's name to the tool's configuration:</p>
        <ul>
            <li>Open the tool's overlay by using <em>Shift + Space</em></li>
            <li>Click the <i class="fas fa-cog align-bottom" /> icon to open the <em>Settings</em> menu</li>
            <li>Go to the <em>Price Check</em> tab</li>
            <li>Enter your PoE account's name in the <em>Account Name</em> textbox</li>
            <li>(Optional) You can also change the targetted <em>League</em></li>
            <li>Click the <em>Save</em> button at the foot of the form</li>
        </ul>
    </div>
    <div v-if="unique" class="layout-column overflow-y-auto overflow-x-hidden">
        <div class="hoverBox" v-html="unique?.data"></div>
    </div>
  </div>
</template>

<script lang="ts">

    interface UniqueItem {
        name: string | null;
        data: string | null;
    }

    import { defineComponent, PropType, shallowRef } from 'vue'
    import { useI18nNs } from '@/web/i18n'
    import { AppConfig } from '@/web/Config'
    import { ItemFilters, StatFilter } from '../price-check/filters/interfaces'
    import { ParsedItem } from '@/parser'

    function isNullOrEmptyString(string: string|null) {
        return string == null || (string ?? '').trim() == '';
    }

    function usePoeLadderApi () {

        const error = shallowRef<string | null>(null)
        const searchResult = shallowRef<UniqueItem | null>(null)

        async function search(
            uniqueName: string|null = null,
            uniqueIdentfier: string|null = null,
            accountName: string|null = null,
            ladderIdentifier: string|null = null,
            uniqueItemBase: string|null = null,
            uniqueItemBaseImplicit: string|null = null,
            uniqueItemMapTier: number|null = null,
            uniqueRawText: string|null = null
        ) {

            // Check for multiple bases
            var disambiguation = 1;
            switch(uniqueIdentfier){
                case "Beacon_of_Madness":

                    disambiguation = (uniqueRawText?.indexOf("chance to deal Double Damage") ?? -1) >= 0 ? 2 : (uniqueRawText?.indexOf("Conflux while affected") ?? -1) >= 0 ? 3 : 1; // 1 = Chaos

                    break;

                case "Combat_Focus":

                    disambiguation = uniqueItemBase == "Viridian Jewel" ? 2 : uniqueItemBase == "Cobalt Jewel" ? 3 : 1; // 1 = Crimson Jewel

                    break;

                case "Doryanis_Delusion":

                    disambiguation = uniqueItemBase == "Slink Boots" ? 2 : uniqueItemBase == "Sorcerer Boots" ? 3 : 1; // 1 = Titan Greaves

                    break;

                case "Grand_Spectrum":

                    switch(uniqueItemBase){

                        case "Cobalt Jewel":

                            disambiguation = (uniqueRawText?.indexOf("Critical Strike Multiplier") ?? -1) >= 0 ? 2 : (uniqueRawText?.indexOf("Critical Strike Chance") ?? -1) >= 0 ? 7 : 9; // 1 = Max Power
                            break;

                        case "Crimson Jewel":
                            
                            disambiguation = (uniqueRawText?.indexOf("Maximum Life") ?? -1) >= 0 ? 5 : (uniqueRawText?.indexOf("Elemental Resistances") ?? -1) >= 0 ? 6 : 1; // 1 = Minimum Endurance
                            break;

                        case "Viridian Jewel":
                            
                            disambiguation = (uniqueRawText?.indexOf("Avoid Elemental Ailments") ?? -1) >= 0 ? 3 : (uniqueRawText?.indexOf("increased Elemental Damage") ?? -1) >= 0 ? 4 : 8; // 8 = Minimum Frenzy
                            break;
                        
                        default:
                            disambiguation = 1;
                    }

                    break;

                case "Impresence":

                    disambiguation = (uniqueRawText?.indexOf("Despair") ?? -1) >= 0 ? 2 : (uniqueRawText?.indexOf("Vulnerability") ?? -1) >= 0 ? 3 : (uniqueRawText?.indexOf("Conductivity") ?? -1) >= 0 ? 4 : (uniqueRawText?.indexOf("Frostbite") ?? -1) >= 0 ? 5 : 1; // 1 = Flammability

                    break;

                case "Lord_of_Steel":

                    disambiguation = (uniqueRawText?.indexOf("chance to Impale Enemies") ?? -1) >= 0 ? 2 : (uniqueRawText?.indexOf("Overwhelms") ?? -1) >= 0 ? 3 : 1; // 1 = Impale Effect

                    break;

                case "Precursors_Emblem":

                    switch(uniqueItemBase){

                        case "Two-Stone Ring":
                            disambiguation = (uniqueRawText?.indexOf("Dexterity and Intelligence") ?? -1) >= 0 ? 2 : (uniqueRawText?.indexOf("Strength and Dexterity") ?? -1) >= 0 ? 3 : 1; // 1 = Strength and Intelligence
                            break;

                        case "Topaz Ring":
                            disambiguation = 4;
                            break;
                        
                        case "Sapphire Ring":
                            disambiguation = 5;
                            break;
                                                    
                        case "Ruby Ring":
                            disambiguation = 6;
                            break;
                                                    
                        case "Prismatic Ring":
                            disambiguation = 7;
                            break;
                        
                        default:
                            disambiguation = 1;
                    }                 

                    break;

                case "The_Beachhead":

                    disambiguation = uniqueItemMapTier == 10 ? 2 : uniqueItemMapTier == 5 ? 3 : 1; // 1 = 15

                    break;

                case "Volkuurs_Guidance":

                    disambiguation = (uniqueRawText?.indexOf("Cold Damage to Spells") ?? -1) >= 0 ? 2 : (uniqueRawText?.indexOf("Fire Damage to Spells") ?? -1) >= 0 ? 3 : 1; // 1 = Lightning Damage to Spells

                    break;

                default:

                    disambiguation = 1;
            }

            var url = `https://poeladder.com/api/v1/users/${accountName}/ladders/${ladderIdentifier}/uniques/${uniqueIdentfier}/?disambiguation=${disambiguation}&nullableResult=1&baseItem=${encodeURIComponent(uniqueItemBase ?? "")}&baseImplicit=${uniqueItemBaseImplicit}`

            if(!isNullOrEmptyString(uniqueIdentfier) && !isNullOrEmptyString(accountName) && !isNullOrEmptyString(ladderIdentifier)){

                fetch(url,  {
                    headers: {
                        'jwt-auth': "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjcmVhdGVkIjoiV2l0aCBsb3ZlIiwic2VjdXJlZCI6IldpdGggc3RpY2t5LXRhcGUiLCJ1cGRhdGVkIjoiRnJvbSB0aW1lIHRvIHRpbWUiLCJ0b2tlbkd1aWQiOiI0ODFjZjQ1OS0xOTRlLTQxMTMtYWQxNy0yMGQ1OGNmMjJlNTUiLCJwYXNzcGhyYXNlIjpudWxsfQ.GjjC079qiOw8RxpUHFF4YLeoudYjijjzbLf4H8saTyQ"
                    }
                })
                .then(async response => {
                    searchResult.value = {
                        name: uniqueName,
                        data: await response.text()
                    };
                    if(isNullOrEmptyString(searchResult.value.data)){
                        searchResult.value.data = `<p>Unique not found in ${ladderIdentifier}</p>`;
                    }
                });
            }
        }

        return { error, searchResult, search };
    }

    export default defineComponent({
        props: {
            filters: {
                type: Object as PropType<ItemFilters>,
                required: true
            },
            stats: {
                type: Array as PropType<StatFilter[]>,
                required: true
            },
            item: {
                type: Object as PropType<ParsedItem>,
                required: true
            }
        },
        computed: {
            accountNameMissing () {
                return (AppConfig().accountName ?? '').trim() == '';
          }
        },
        setup (props) {

            const { error, searchResult, search } = usePoeLadderApi();
            const { t } = useI18nNs('trade_result');

            return {
                t,
                unique: searchResult,
                execSearch: async () => {

                    if(props.item.info.namespace == "UNIQUE"){

                        const uniqueName = props.item.info.name;
                        const uniqueIdentfier = decodeURIComponent(uniqueName).replace(/[^a-zA-Z0-9_\s]/g,'').replace(/\s/g, "_");
                        const uniqueItemBase = props.item.info.unique?.base;
                        const uniqueItemBaseImplicit = encodeURIComponent(props.item.statsByType.filter(function (stat) {
                            return stat.type == "implicit";
                        })[0]?.stat?.ref ?? "");
                        const uniqueItemMapTier = props.item.mapTier;
                        const uniqueRawText = props.item.rawText;
                        const ladderIdentifier = "SSF_" + AppConfig().leagueId;
                        const accountName = AppConfig().accountName;

                        await search(uniqueName, uniqueIdentfier, accountName, ladderIdentifier, uniqueItemBase, uniqueItemBaseImplicit, uniqueItemMapTier, uniqueRawText);

                    } else {

                        await search();
                    }
                },
                error
            };
        }
    });
</script>

<style lang="postcss">

    /* Fonts */

    /* https://web.poecdn.com/protected/css/font.css?v=5931036dae40633eaf1680b366588c43&key=4rcNCimGiCj3Ztjc_v1BUg */
    @font-face {
        font-family: "FontinSmallCaps";
        font-weight: normal;
        font-style: normal;
        src: url('https://web.poecdn.com/font/fontin-smallcaps-webfont.woff') format("woff")
    }

    /* Help Text */
    .helpText ul {
        list-style-type: square;
        margin-bottom: 15px;
    }
    
    .helpText ul li {
        margin-left: 25px;
    }

    .helpText ul li i {
        position: relative;
        top: -3px;
    }

    /* Uniques Hover */

    .hoverBox {
        min-width: 430px;
        max-width: 430px;
        text-align: center;
        font-family: FontinSmallCaps,Verdana,Arial,Helvetica,sans-serif;
        color: #ffffffde;
        line-height: 1.5;
        font-weight: 400;
        color-scheme: light dark;
        background-color: #131313;
        border: 1px solid rgb(19,9,0);
        font-synthesis: none;
        text-rendering: optimizeLegibility;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        -webkit-text-size-adjust: 100%;
    }

    .hoverBox .uniqueHover .banner {
        background: url(https://web.poecdn.com/protected/image/item/popup/header-double-unique-left.png?v=1682897765566&key=wWot3i-c3fi9mITi48Vl9g) top left no-repeat,url(https://web.poecdn.com/protected/image/item/popup/header-double-unique-right.png?v=1682897765566&key=-dfCEOleavUZZdgllVgSsQ) top right no-repeat,url(https://web.poecdn.com/protected/image/item/popup/header-double-unique-middle.png?v=1682897765566&key=eZIr40k4fczlgGEz5heqLA) top center repeat-x;
        color: #af6025;
        height: 52px;
    }

    .hoverBox .uniqueHover .banner::before,
    .hoverBox .uniqueHover .banner::after {
        height: 25px;
        width: 25px;
        margin: -25px;
        position: relative;
        display: block;
        content: "";
        background-repeat: no-repeat;
    }

    .hoverBox .uniqueHover .banner::before {
        top: 39px;
        left: 30px;
        float: left;
    }

    .hoverBox .uniqueHover .banner::after {
        top: -12px;
        right: 31px;
        float: right;
    }

    .hoverBox .uniqueHover .banner.crusader::before,
    .hoverBox .uniqueHover .banner.crusader::after,
    .hoverBox .uniqueHover .banner.crusader::before,
    .hoverBox .uniqueHover .banner.crusaderelder::after,
    .hoverBox .uniqueHover .banner.crusaderhunter::before,
    .hoverBox .uniqueHover .banner.crusaderredeemer::before,
    .hoverBox .uniqueHover .banner.crusadershaper::after,
    .hoverBox .uniqueHover .banner.crusaderwarlord::before {
        background-image: url(https://web.poecdn.com/protected/image/item/popup/crusader-symbol.png?v=1689575801910&key=dcLGV7cDJf-dulB8P_Q-yQ);
    }

    .hoverBox .uniqueHover .banner.hunter::before,
    .hoverBox .uniqueHover .banner.hunter::after,
    .hoverBox .uniqueHover .banner.crusaderhunter::after,
    .hoverBox .uniqueHover .banner.elderhunter::after,
    .hoverBox .uniqueHover .banner.hunterredeemer::before,
    .hoverBox .uniqueHover .banner.huntershaper::after,
    .hoverBox .uniqueHover .banner.hunterwarlord::before {
        background-image: url(https://web.poecdn.com/protected/image/item/popup/hunter-symbol.png?v=1689575801954&key=x7-F-t4jlxzMd1eJOUSafA);
    }

    .hoverBox .uniqueHover .banner.redeemer::before,
    .hoverBox .uniqueHover .banner.redeemer::after,
    .hoverBox .uniqueHover .banner.crusaderredeemer::after,
    .hoverBox .uniqueHover .banner.elderredeemer::after,
    .hoverBox .uniqueHover .banner.hunterredeemer::after,
    .hoverBox .uniqueHover .banner.redeemershaper::after,
    .hoverBox .uniqueHover .banner.redeemerwarlord::before {
        background-image: url(https://web.poecdn.com/protected/image/item/popup/redeemer-symbol.png?v=1689575801958&key=nEc-TPctqwszY696cmlaZQ);
    }

    .hoverBox .uniqueHover .banner.warlord::before,
    .hoverBox .uniqueHover .banner.warlord::after,
    .hoverBox .uniqueHover .banner.crusaderwarlord::after,
    .hoverBox .uniqueHover .banner.elderwarlord::after,
    .hoverBox .uniqueHover .banner.hunterwarlord::after,
    .hoverBox .uniqueHover .banner.redeemerwarlord::after,
    .hoverBox .uniqueHover .banner.shaperwarlord::after { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/warlord-symbol.png?v=1689575802134&key=76scs3jk3FrVGOLHdM5AuA);
    }

    .hoverBox .uniqueHover .banner.veiled::before,
    .hoverBox .uniqueHover .banner.veiled::after { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/veiled-symbol.png?v=1682897765714&key=FQrrJgHpP4bo1Im-sOGTcA);
    }

    .hoverBox .uniqueHover .banner.synthesised::before,
    .hoverBox .uniqueHover .banner.synthesised::after { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/synthetic-symbol.png?v=1682897765702&key=CUVu3Zqxrzd1CwWcIH-6xg);
    }

    .hoverBox .uniqueHover .banner.replica::before,
    .hoverBox .uniqueHover .banner.replica::after,
    .hoverBox .uniqueHover .banner.elderreplica::before { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/experimented-symbol.png?v=1682897765562&key=dWh87tRCnnyVEMYjjf6RbQ);
    }

    .hoverBox .uniqueHover .banner.shaper::before,
    .hoverBox .uniqueHover .banner.shaper::after,
    .hoverBox .uniqueHover .banner.eldershaper::before,
    .hoverBox .uniqueHover .banner.crusadershaper::before,
    .hoverBox .uniqueHover .banner.huntershaper::before,
    .hoverBox .uniqueHover .banner.redeemershaper::before,
    .hoverBox .uniqueHover .banner.shaperwarlord::before { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/shaper-symbol.png?v=1682897765698&key=PQH0ATL4n268OngTnsQ0MQ);
    }

    .hoverBox .uniqueHover .banner.elder::before,
    .hoverBox .uniqueHover .banner.elder::after,
    .hoverBox .uniqueHover .banner.crusaderelder::before,
    .hoverBox .uniqueHover .banner.elderhunter::before,
    .hoverBox .uniqueHover .banner.elderredeemer::before,
    .hoverBox .uniqueHover .banner.elderwarlord::before,
    .hoverBox .uniqueHover .banner.elderreplica::after,
    .hoverBox .uniqueHover .banner.eldershaper::after { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/elder-symbol.png?v=1682897765506&key=YS6Fsnxyuc8gpqfPz_mykw);
    }

    .hoverBox .uniqueHover .banner.searing::before,
    .hoverBox .uniqueHover .banner.searing::after,
    .hoverBox .uniqueHover .banner.searingtangled::before { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/searing-symbol.png?v=1682897765698&key=iiOWJvWHj8-0bH0e28X7Xw);
    }

    .hoverBox .uniqueHover .banner.tangled::before,
    .hoverBox .uniqueHover .banner.tangled::after,
    .hoverBox .uniqueHover .banner.searingtangled::after { 
        background-image: url(https://web.poecdn.com/protected/image/item/popup/tangled-symbol.png?v=1682897765702&key=b-SVlhPy2jy-UJSTBaV2Cw);
    }

    .hoverBox .uniqueHover .banner .name {
        padding-top: 8px;
        white-space: nowrap;
    }

    .hoverBox .uniqueHover .banner .baseItem {
        margin-top: -1px;
        white-space: nowrap;
    }

    .hoverBox .uniqueHover .detail {
        padding: 0.4em 0em 0.5em 0em;
        vertical-align: baseline;
        font-size: 70%;
    }

    .hoverBox .uniqueHover .detail .prop {
        color: rgb(127, 127, 127);
    }

    .hoverBox .uniqueHover .detail em {
        color: #88f;
    }

    .hoverBox .uniqueHover .detail strong {
        color: #fff;
    }

    .hoverBox .uniqueHover .detail .corr::after {
        color: #d20000;
        content: "Corrupted";
    }

    .hoverBox .uniqueHover .detail div {
        min-width: 430px;
        max-width: 430px;
        width: max-content; /* Allow flexible wrapping of text up to max width */
        width: -webkit-max-content; /* Add fix for cutycapt clipping the foot of the image when it fails to resize to content */
        padding: 0 5px; /* Prevent text from banging up against the hover box */
        box-sizing: border-box; /* Allow these elements to resize the hover box */
        -webkit-box-sizing: border-box;
        margin: 0 auto; /* Make sure these items are centered in the hover box when there is a size disparity */
    }

    .hoverBox .uniqueHover .detail .ench {
        color: #b4b4ff;
    }

    .hoverBox .uniqueHover .detail .sco {
        color: #ff6e25;
    }

    .hoverBox .uniqueHover .detail .impl {
        color: #88f;
    }

    .hoverBox .uniqueHover .detail .mod {
        color: #88f;
    }

    .hoverBox .uniqueHover .detail .mod em, .hoverBox .uniqueHover .detail .impl em {
        color: #fff;
    }

    .hoverBox .uniqueHover .detail .mod .desc {
        color: rgb(127, 127, 127);
    }

    .hoverBox .uniqueHover .detail .cru {
        color: #ff7339;
    }

    .hoverBox .uniqueHover .detail .cru .cruTree {
        margin: 5px;
    }

    .hoverBox .uniqueHover .detail .flav {
        font-style: italic;
        color: rgb(175, 96, 37);
    }

    .hoverBox .uniqueHover .detail .flav em {
        color: #7f7f7f;
    }

    .hoverBox .uniqueHover .detail .scoLvl {
        color: #ff6e25;
    }

    .hoverBox .uniqueHover .detail i.veiled::after {
        height: 20px;
        display: block;
        margin: 0 auto;
        content: "";
        animation: animate-veiled-mod 3s steps(90) reverse infinite;
        will-change: background-position;
        white-space: nowrap;
        background-image: url(https://web.poecdn.com/protected/image/item/veiled/prefix_05.png?v=1682897766058&key=iO-4nU1hcIciOa47-d1H6Q);
        background-position-x: -14220px;
        background-position-y: 50%;
        width: 158px;
        will-change: "background-position";
    }

    @keyframes animate-veiled-mod {
        100% {
            background-position: 0;
        }
    }

    .hoverBox .uniqueHover .detail .ench::before,
    .hoverBox .uniqueHover .detail .req::before,
    .hoverBox .uniqueHover .detail .sco::before,
    .hoverBox .uniqueHover .detail .impl::before,
    .hoverBox .uniqueHover .detail .mod::before,
    .hoverBox .uniqueHover .detail .corr::before,
    .hoverBox .uniqueHover .detail .cru::before,
    .hoverBox .uniqueHover .detail .flav::before,
    .hoverBox .uniqueHover .detail .scoLvl::before {
        background: url(https://web.poecdn.com/protected/image/item/popup/separator-unique.png?v=1690324640931&key=KxJv36tBZx3tAvjfi8x4vQ) center no-repeat;
        height: 10px;
        display: block;
        content: "";
    }

</style>