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
    <div v-if="!accountName">
        <p>Please add your Account Name to the tool's configuration:</p>
        <ul>
            <li>Open the <strong>Settings</strong> menu with <strong>Shift + Space</strong></li>
            <li>Go to the <strong>Price Check</strong> tab</li>
            <li>Enter your PoE account's name in the <strong>Account Name</strong> textbox</li>
            <li>Click the <strong>Save> button at the foot of the form</strong></li>
        </ul>
    </div>
    <div v-if="unique" class="layout-column overflow-y-auto overflow-x-hidden">
        <div v-html="unique.data"></div>
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

    function usePoeLadderApi () {

        const error = shallowRef<string | null>(null)
        const searchResult = shallowRef<UniqueItem | null>(null)

        async function search (uniqueName: string|null = null, uniqueIdentfier: string|null = null, accountName: string|null = null, ladderIdentifier: string|null = null) {

            var url = `https://poeladder.com/api/v1/users/${accountName}/ladders/${ladderIdentifier}/uniques/${uniqueIdentfier}/?disambiguation=1`
console.log(url);
            if((uniqueIdentfier ?? "").trim() != "" &&  (accountName ?? "").trim() != "" && (ladderIdentifier ?? "").trim() != ""){

                fetch(url,  {
                    headers: {
                        'jwt-auth': "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjcmVhdGVkIjoiV2l0aCBsb3ZlIiwic2VjdXJlZCI6IldpdGggc3RpY2t5LXRhcGUiLCJ1cGRhdGVkIjoiRnJvbSB0aW1lIHRvIHRpbWUiLCJ0b2tlbkd1aWQiOiI4NmE1NWZmMC00ODc5LTQzNmUtYWYzZi1jNmNmZThhNDkxZWQiLCJwYXNzcGhyYXNlIjpudWxsfQ.ZA4C4JGZogyjH4aFatKUCE0e3G3b6WJ73MZ3lZd36XY"
                    }
                })
                .then(async response => {
                    searchResult.value = {
                        name: uniqueName,
                        data: await response.text()
                    };
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
        setup (props) {

            const { error, searchResult, search } = usePoeLadderApi();
            const { t } = useI18nNs('trade_result');

            return {
                t,
                unique: searchResult,
                accountName: AppConfig().accountName,
                execSearch: async () => {

                    if(props.item.info.namespace == "UNIQUE"){

                        const uniqueName = props.item.info.name;
                        const uniqueIdentfier = decodeURIComponent(uniqueName).replace(/[^a-zA-Z0-9_\s]/g,'').replace(/\s/g, "_");
                        const ladderIdentifier = "SSF_" + AppConfig().leagueId;
                        const accountName = AppConfig().accountName;

                        await search(uniqueName, uniqueIdentfier, accountName, ladderIdentifier);

                    } else {

                        await search();
                    }
                },
                error
            };
        }
    });
</script>