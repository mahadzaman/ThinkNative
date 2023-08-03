<template>
    <div class="modal fade" id="editNameModal" role="dialog" aria-hidden="true">
        <div class="modal-dialog taskEditor" role="document">
            <div class="modal-content">
                <div class=" modalEditor taskEditorModal">
                    <div class="d-flex justify-content-between align-center">
                        <h5 class="modal-title modalEditorTitle">
                            <span>Edit Group Name</span>
                        </h5>
                        <button
                            type="button"
                            class="close CloseBtn"
                            data-dismiss="modal"
                            aria-label="Close"
                        >
                            <i class="icon-close"></i>
                        </button>
                    </div>
                </div>

                <ValidationObserver v-slot = "{ passes, reset }">
                    <form @submit.prevent = "passes(updateThreadName)" @reset.prevent = "reset" ref = "form">
                <div class="modal-body">
<!--                    <input type="text" class="form-control thread" placeholder="Name this Group Thread" :value="getThreadName()">-->
                    <FormElements rules = "required" elem = "input"
                                  placeholder = "Name this Group Thread"
                                  :maxlength="36"
                                  :value = "getThreadName()" name = "thread name"
                                  v-on:val = "bindValue" />
                </div>

                <div class="modal-footer-padding">
                    <div class="raw-flex just-end">
                        <button type = "submit" class="btn btn-save-note" :disabled="formData.thread_name === ''">Save</button>
                        <!-- <button type="reset" class="hide resetForm">
                            Reset
                        </button> -->
                    </div>
                </div>
                    </form>
                </ValidationObserver>
            </div>
        </div>
    </div>
</template>

<script>
import FormElements from '../../components/partials/FormElements';
import axios from "../../store/axios";
import to from "await-to-js";
import service from "../../services/groups";
export default {
    name: "EditName",
    components : {FormElements},
    props: ["threadName", "isDefaultName"],
    data () {
        return {
            formData : {
                thread_name : '',
            },
        }
    },
    methods: {
        getThreadName () {
            return (this.isDefaultName) ? '' : this.threadName;
        },
        bindValue ( obj ) {
            this.formData[ obj.ref ] = obj.value;
        },
        updateThreadName: function () {
            this.$emit('updateThreadName', this.formData.thread_name);
        }
    }
};
</script>

<style lang="scss" scoped>
.modal-body {
    padding: 20px 40px;
}
</style>
