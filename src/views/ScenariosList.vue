<template>
    <div class='scenarios-list'>
        <v-data-table
            :headers='headers'
            :items='scenarios'
            :items-per-page='5'
            v-on:click:row='navigateToScenario'
        >
            <template v-slot:top>
                <v-toolbar
                    flat
                >
                    <v-toolbar-title>{{ $tc('generic.scenario', 2) }}</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-dialog
                        v-model='dialog'
                        max-width='500px'
                    >
                        <template v-slot:activator='{ on, attrs }'>
                            <v-btn
                                color='primary'
                                dark
                                class='mb-2'
                                v-bind='attrs'
                                v-on='on'
                            >
                                {{ $t('scenarios.new') }}
                            </v-btn>
                        </template>
                        <v-card>
                            <v-card-title>
                                <span class='headline'>{{ formTitle }}</span>
                            </v-card-title>

                            <v-card-text>
                                <v-container>
                                    <v-row>
                                        <v-col
                                            cols='12'
                                            sm='12'
                                            md='12'
                                        >
                                            <v-text-field
                                                v-model='editedItem.name'
                                                :label="[$tc('generic.scenario', 1), $t('generic.name')].join(' ')"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col
                                            cols='12'
                                            sm='12'
                                            md='12'
                                        >
                                            <v-textarea
                                                v-model='editedItem.description'
                                                :label="$t('generic.description')"
                                            ></v-textarea>
                                        </v-col>
                                        <v-col
                                            cols='12'
                                            sm='6'
                                            md='6'
                                        >
                                            <v-select
                                                v-model='editedItem.impact'
                                                :label="$t('generic.impact')"
                                                :items='impactLevels'
                                            >
                                            </v-select>
                                        </v-col>
                                        <v-col
                                            cols='12'
                                            sm='6'
                                            md='6'
                                        >
                                            <v-select
                                                v-model='editedItem.risk'
                                                :label="$t('generic.risk')"
                                                :items='riskLevels'
                                            ></v-select>
                                        </v-col>
                                        <v-col
                                            cols='12'
                                            sm='6'
                                            md='6'
                                        >
                                            <v-select
                                                v-model='editedItem.areaOfEffect'
                                                :label="$t('generic.area-of-effect')"
                                                :items='areaOfEffect'></v-select>
                                        </v-col>
                                    </v-row>
                                </v-container>
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn
                                    color='blue darken-1'
                                    text
                                    @click='close'
                                >
                                    {{ $t('actions.cancel') }}
                                </v-btn>
                                <v-btn
                                    color='blue darken-1'
                                    text
                                    @click='save'
                                >
                                    {{ $t('actions.save') }}
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                    <v-dialog v-model='dialogDelete' max-width='500px'>
                        <v-card>
                            <v-card-title class='headline'>Are you sure you want to delete this item?</v-card-title>
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color='blue darken-1' text @click='closeDelete'>Cancel</v-btn>
                                <v-btn color='blue darken-1' text @click='deleteItemConfirm'>OK</v-btn>
                                <v-spacer></v-spacer>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:item.actions='{ item }'>
                <v-icon
                    small
                    class='mr-2'
                    @click='editItem(item)'
                >
                    mdi-pencil
                </v-icon>
                <v-icon
                    small
                    @click='deleteItem(item)'
                >
                    mdi-delete
                </v-icon>
            </template>
            <template v-slot:no-data>
                {{ $t('scenarios.no-rows') }}
            </template>
        </v-data-table>
    </div>
</template>

<script>
import { getRepository } from '@/util/repositories';
import Scenario, { SCENARIO_ENDPOINT } from '@/models/Scenario';

export default {
    name: 'ScenariosList',
    data() {
        return {
            scenarios: [],
            headers: [
                {
                    text: 'Name',
                    value: 'name',
                },
                {
                    text: 'Impact',
                    value: 'impact',
                },
                {
                    text: 'Risk',
                    value: 'risk',
                },
                {
                    text: 'Area of Effect',
                    value: 'areaOfEffect',
                },
                {
                    text: 'Actions',
                    value: 'actions',
                    sortable: false,
                },
            ],
            error: null,
            dialog: false,
            dialogDelete: false,
            editedIndex: -1,
            editedItem: {
                name: '',
                description: '',
                impact: 'MINIMAL',
                risk: 'LOW',
                id: '',
                areaOfEffect: 'PERSONAL',
            },
            defaultItem: {
                name: '',
                description: '',
                impact: 'MINIMAL',
                risk: 'LOW',
                areaOfEffect: 'PERSONAL',
                id: '',
            },
            impactLevels: [
                'MINIMAL',
                'MINOR',
                'SIGNIFICANT',
                'MAJOR',
                'TEOTWAWKI',
            ],
            areaOfEffect: [
                'PERSONAL',
                'FAMILY',
                'COMMUNITY',
                'REGION',
                'NATION',
                'WORLDWIDE',
            ],
            riskLevels: [
                'LOW',
                'MEDIUM',
                'HIGH',
            ],
        };
    },
    computed: {
        formTitle() {
            return this.editedIndex === -1 ? 'New Scenario' : 'Edit Scenario';
        },
    },
    created() {
        const scenariosRepository = getRepository(SCENARIO_ENDPOINT, Scenario);
        scenariosRepository.findAll()
            .then(scenarios => {
                console.log(scenarios);
                this.scenarios = scenarios;
            })
            .catch(e => {
                this.error = e;
            });
    },
    methods: {
        close() {
            this.dialog = false;
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            });
        },
        save() {
            const scenariosRepository =
                getRepository(SCENARIO_ENDPOINT, Scenario);

            const onSave = scenario => {
                if (this.scenarios.find(x => x.id === scenario.id)) {
                    Object.assign(
                        this.scenarios.find(x => x.id === scenario.id),
                        scenario,
                    );
                } else {
                    this.scenarios.push(scenario);
                }
                this.close();
            };

            if (this.editedItem.id !== '') {
                scenariosRepository.update(this.editedItem.id, this.editedItem).then(onSave);
            } else {
                scenariosRepository.create(this.editedItem).then(onSave);
            }
        },
        editItem(item) {
            this.editedIndex = this.scenarios.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.dialog = true;
        },
        deleteItem(item) {
            this.editedIndex = this.scenarios.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.dialogDelete = true;
        },
        closeDelete() {
            this.dialogDelete = false;
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            });
        },
        deleteItemConfirm() {
            const scenariosRepository =
                getRepository(SCENARIO_ENDPOINT, Scenario);
            scenariosRepository.delete(this.editedItem.id);

            this.scenarios.splice(this.editedIndex, 1);
            this.closeDelete();
        },
        navigateToScenario(evt, { item }) {
            console.log(item);
        },
    },
};
</script>

<style scoped>
.scenarios-list {
    padding: 2rem;
}
</style>
