<template>
    <BasePage :title="'Inverter Settings'" :isLoading="dataLoading">
        <BootstrapAlert v-model="alert.show" dismissible :variant="alert.type">
            {{ alert.message }}
        </BootstrapAlert>

        <div class="card">
            <div class="card-header text-bg-primary">Add a new Inverter</div>
            <div class="card-body">
                <form class="form-inline" v-on:submit.prevent="onSubmit">
                    <div class="form-group">
                        <label>Serial</label>
                        <input v-model="newInverterData.serial" type="number" class="form-control ml-sm-2 mr-sm-4 my-2"
                            required />
                    </div>
                    <div class="form-group">
                        <label>Name</label>
                        <input v-model="newInverterData.name" type="text" class="form-control ml-sm-2 mr-sm-4 my-2"
                            maxlength="31" required />
                    </div>
                    <div class="ml-auto text-right">
                        <button type="submit" class="btn btn-primary my-2">Add</button>
                    </div>
                    <div class="alert alert-secondary" role="alert">
                        <b>Hint:</b> You can set additional parameters after you have created the inverter. Use the pen
                        icon in the inverter list.
                    </div>
                </form>
            </div>
        </div>

        <div class="card mt-5">
            <div class="card-header text-bg-primary">Inverter List</div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table">
                        <thead>
                            <tr>
                                <th scope="col">Serial</th>
                                <th>Name</th>
                                <th>Type</th>
                                <th>Action</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="inverter in sortedInverters" v-bind:key="inverter.id">
                                <td>{{ inverter.serial }}</td>
                                <td>{{ inverter.name }}</td>
                                <td>{{ inverter.type }}</td>
                                <td>
                                    <a href="#" class="icon text-danger" title="Delete inverter">
                                        <BIconTrash v-on:click="onOpenModal(modalDelete, inverter)" />
                                    </a>&nbsp;
                                    <a href="#" class="icon" title="Edit inverter">
                                        <BIconPencil v-on:click="onOpenModal(modal, inverter)" />
                                    </a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </BasePage>

    <div class="modal" id="inverterEdit" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Edit Inverter</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="mb-3">
                            <label for="inverter-serial" class="col-form-label">Inverter Serial:</label>
                            <input v-model="selectedInverterData.serial" type="number" id="inverter-serial"
                                   class="form-control" />
                            <label for="inverter-name" class="col-form-label">Inverter Name:</label>
                            <input v-model="selectedInverterData.name" type="text" id="inverter-name"
                                   class="form-control" maxlength="31" />
                        </div>

                        <div v-for="(max, index) in selectedInverterData.channel" :key="`${index}`">
                            <div class="row g-2">
                                <div class="col-md">
                                    <label :for="`inverter-name_${index}`" class="col-form-label">Name string {{ index +1 }}:</label>
                                    <div class="d-flex mb-2">
                                        <div class="input-group">
                                            <input type="text" class="form-control" :id="`inverter-name_${index}`" maxlength="31"
                                                v-model="selectedInverterData.channel[index].name" />
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-5">
                                    <label :for="`inverter-max_${index}`" class="col-form-label">Max power string {{ index +1 }}:</label>
                                    <div class="d-flex mb-2">
                                        <div class="input-group">
                                            <input type="number" class="form-control" :id="`inverter-max_${index}`" min="0"
                                                v-model="selectedInverterData.channel[index].max_power"
                                                :aria-describedby="`inverter-maxDescription_${index} inverter-customizer`" />
                                            <span class="input-group-text" :id="`inverter-maxDescription_${index}`">W<sub>p</sub><sup>*</sup></span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div :id="`inverter-customizer`" class="form-text">*) Input the W<sub>p</sub> of the channel to
                            calculate irradiation.</div>
                    </form>

                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" @click="onCloseModal(modal)"
                        data-bs-dismiss="modal">Cancel</button>
                    <button type="button" class="btn btn-primary" @click="onEditSubmit">Save
                        changes</button>
                </div>
            </div>
        </div>
    </div>

    <div class="modal" id="inverterDelete" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Delete Inverter</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    Are you sure you want to delete the inverter "{{ selectedInverterData.name }}" with serial number
                    {{ selectedInverterData.serial }}?
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" @click="onCloseModal(modalDelete)"
                        data-bs-dismiss="modal">Cancel</button>
                    <button type="button" class="btn btn-danger" @click="onDelete">Delete</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import BasePage from '@/components/BasePage.vue';
import {
    BIconTrash,
    BIconPencil
} from 'bootstrap-icons-vue';
import * as bootstrap from 'bootstrap';
import BootstrapAlert from "@/components/BootstrapAlert.vue";
import { handleResponse, authHeader } from '@/utils/authentication';

declare interface Channel {
    name: string;
    max_power: number;
}

declare interface Inverter {
    id: string;
    serial: number;
    name: string;
    type: string;
    channel: Array<Channel>;
}

declare interface AlertResponse {
    message: string;
    type: string;
    show: boolean;
}

export default defineComponent({
    components: {
        BasePage,
        BootstrapAlert,
        BIconTrash,
        BIconPencil,
    },
    data() {
        return {
            modal: {} as bootstrap.Modal,
            modalDelete: {} as bootstrap.Modal,
            newInverterData: {} as Inverter,
            selectedInverterData: {} as Inverter,
            inverters: [] as Inverter[],
            dataLoading: true,
            alert: {} as AlertResponse
        };
    },
    mounted() {
        this.modal = new bootstrap.Modal('#inverterEdit');
        this.modalDelete = new bootstrap.Modal('#inverterDelete');
    },
    created() {
        this.getInverters();
    },
    computed: {
        sortedInverters(): Inverter[] {
            return this.inverters.slice().sort((a, b) => {
                return a.serial - b.serial;
            });
        },
    },
    methods: {
        getInverters() {
            this.dataLoading = true;
            fetch("/api/inverter/list", { headers: authHeader() })
                .then((response) => handleResponse(response, this.$emitter, this.$router))
                .then((data) => {
                    this.inverters = data.inverter;
                    this.dataLoading = false;
                });
        },
        callInverterApiEndpoint(endpoint: string, jsonData: string) {
            const formData = new FormData();
            formData.append("data", jsonData);

            fetch("/api/inverter/" + endpoint, {
                method: "POST",
                headers: authHeader(),
                body: formData,
            })
                .then((response) => handleResponse(response, this.$emitter, this.$router))
                .then((data) => {
                    this.getInverters();
                    this.alert = data;
                    this.alert.show = true;
                });
        },
        onSubmit() {
            this.callInverterApiEndpoint("add", JSON.stringify(this.newInverterData));
            this.newInverterData = {} as Inverter;
        },
        onDelete() {
            this.callInverterApiEndpoint("del", JSON.stringify({ id: this.selectedInverterData.id }));
            this.onCloseModal(this.modalDelete);
        },
        onEditSubmit() {
            this.callInverterApiEndpoint("edit", JSON.stringify(this.selectedInverterData));
            this.onCloseModal(this.modal);
        },
        onOpenModal(modal: bootstrap.Modal, inverter: Inverter) {
            // deep copy inverter object for editing/deleting
            this.selectedInverterData = JSON.parse(JSON.stringify(inverter)) as Inverter;
            modal.show();
        },
        onCloseModal(modal: bootstrap.Modal) {
            modal.hide();
        }
    },
});
</script>