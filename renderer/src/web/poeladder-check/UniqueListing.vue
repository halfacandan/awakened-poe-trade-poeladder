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
    <div v-if="unique">
        <p>{{ unique.data }}</p>
    </div>
    <div class="layout-column overflow-y-auto overflow-x-hidden">
        <p>This is a test3</p>
    </div>
  </div>
</template>

<script lang="ts">

    interface UniqueItem {
        name: string;
        data: string;
    }

    import { defineComponent, computed, PropType, shallowRef } from 'vue'
    import { useI18nNs } from '@/web/i18n'
    import { PricingResult } from '../price-check/trade/pathofexile-trade'
    import { ItemFilters, StatFilter } from '../price-check/filters/interfaces'
    import { ParsedItem } from '@/parser'
    import OnlineFilter from '../price-check/trade/OnlineFilter.vue'
    import TradeLinks from '../price-check/trade/TradeLinks.vue'

    function useTradeApi () {

        const error = shallowRef<string | null>(null)
        const searchResult = shallowRef<UniqueItem | null>(null)
        const fetchResults = shallowRef<PricingResult[]>([])

        const groupedResults = computed(() => {

            const out: Array<PricingResult & { listedTimes: number }> = []
            for (const result of fetchResults.value) {
                if (result == null) break
                if (out.length === 0) {
                    out.push({ listedTimes: 1, ...result })
                    continue
                }
                const existingRes = out.find((added, idx) =>
                    (
                        added.accountName === result.accountName &&
                        added.priceCurrency === result.priceCurrency &&
                        added.priceAmount === result.priceAmount
                    ) ||
                    (
                        added.accountName === result.accountName &&
                        (out.length - idx) <= 2 // last or prev
                    )
                )
                if (existingRes) {
                    if (existingRes.stackSize) {
                        existingRes.stackSize += result.stackSize!
                    } else {
                        existingRes.listedTimes += 1
                    }
                } else {
                    out.push({ listedTimes: 1, ...result })
                }
            }
            return out
        })

        async function search (item: ParsedItem) {
            
            if(item.info.namespace == "UNIQUE"){
                console.log(item.info.name);
                searchResult.value = {
                    name: item.info.name,
                    data: "this is a test of data"
                };
                return await item.info.name + " - " + item.info.unique;
            }

            return null;
        }

        return { error, searchResult, groupedResults, search };
    }

    export default defineComponent({
        components: { OnlineFilter, TradeLinks },
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
        setup (props) {

            const { error, searchResult, search } = useTradeApi();
            const { t } = useI18nNs('trade_result');

            return {
                t,
                unique: searchResult,
                execSearch: () => { 
                    search(props.item)
                },
                error
            };
        }
    });
</script>