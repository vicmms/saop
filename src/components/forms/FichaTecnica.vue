<template>
    <div>
        <h4 class="view-name">{{ viewName }}</h4>
        <hr class="red" />
        <div class="row app-options-bar">
            <div class="d-flex align-items-center buttons-component align-items-center">
                <div class="col-md-8">
                    <button title="Crear" class="dt-button btn btn-primary active" type="button" :disabled="readOnlyView"
                        v-if="props.data" @click="readOnlyView = false">
                        <span><b>Editar</b></span>
                    </button>
                </div>
                <AccionesCartera />
            </div>
        </div>
        <div class="d-flex">
            <div class="col-6">
                <InputText v-model="data.clave_compromiso" title="Clave Compromiso:" placeholder="Clave Compromiso"
                    name="clave_compromiso" id="clave_compromiso" :error="errors" class="col-sm-12" />
                <SelectComponent v-model="data.tipo_obra" title="Tipo de Obra:" placeholder="Tipo de Obra" name="tipo_obra"
                    id="tipo_obra" :error="errors" :options="arrayDataTipoObra.data" :optionText="'descripcion'"
                    class="col-sm-12" />
                <InputText v-model="data.requerimiento_de_hacienda" title="Requerimiento de Hacienda:"
                    placeholder="Requerimiento" name="requerimiento_de_hacienda" id="requerimiento_de_hacienda"
                    :error="errors" class="col-sm-12" />
                <SelectComponent v-model="data.tipo_documento" title="Tipo Documento:" placeholder="Tipo Documento"
                    name="tipo_documento" id="tipo_documento" :error="errors" :options="arrayDataTipoDocumento.data"
                    :optionText="'nombre'" class="col-sm-12" />
                <InputText v-model="data.aforo" title="Aforo:" placeholder="Aforo" name="aforo" id="aforo" :error="errors"
                    class="col-sm-12" />
            </div>
            <div class="col-6">
                <label for="nombre-operacion">Documento</label>
                <input data-documento type="file" id="precidencial-prioridad" placeholder="" autocomplete="off"
                    ref="fileInput" accept="application/pdf, image/*" @change="handleFileUpload" />
                <small class="form-text text-muted ml-5" v-if="fileName">{{ fileName[fileName.length - 1] }}</small>
                <small id="descripcion-nivel-small" class="form-text text-muted app-validation"
                    v-if="errors && errors.documento">{{ errors.documento }}</small>
            </div>
        </div>
        <div class="d-flex">
            <div class="col-12">
                <TextAraComponent v-model="data.situacion_actual" :title="'Situacion Actual:'"
                    :placeholder="'Ingresa la situacion...'" :name="'situacion_actual'" :error="errors"
                    :id="'situacion_actual'" class="col-sm-12" />
                <TextAraComponent v-model="data.autorizacion_y_permisos" :title="'Autorizaciones y Permisos:'"
                    :placeholder="'Ingresa autorizacion y permisos...'" :name="'autorizacion_y_permisos'" :error="errors"
                    :id="'autorizacion_y_permisos'" class="col-sm-12" />
                <TextAraComponent v-model="data.alcance_del_proyecto" :title="'Alcance del Proyecto:'"
                    :placeholder="'Ingresa un alcance del proyecto...'" :name="'alcance_del_proyecto'" :error="errors"
                    :id="'alcance_del_proyecto'" class="col-sm-12" />
            </div>
        </div>
        <div class="d-flex col-12">
            <div class="text-right pr-2 flex-grow-1 mt-5" v-if="!readOnlyView">
                <button type="button" class="btn btn-secondary mr-4" @click="handleCancel">
                    Cancelar
                </button>
                <button title="Crear" class="dt-button btn btn-primary active btn-crear" type="button"
                    @click="handleSubmit()">
                    <span v-if="props.data"><b>Guardar</b></span>
                    <span v-else><i class="bi bi-plus"></i> <b>Crear</b></span>
                </button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from "vue";
import AccionesCartera from "@/components/AccionesCarteraPoyectos.vue";
import SelectComponent from "@/components/SelectComponent.vue";
import InputText from "@/components/InputText.vue";
import TextAraComponent from "@/components/TextAraComponent.vue";
import { useForm } from "@/composables/useForm";
import { fichaTecnicaValidations } from "@/utils/validations/fichaTecnica";
import { IFichaTecnica, defaultValues } from "@/utils/models/fichaTecnica";
import usePetition from "@/composables/usePetition";
//Consultas para los selects

const { arrayData: arrayDataTipoObra, getDatas: getDatasTipoObra } =
    usePetition("cat_tipo_obra/");
const { arrayData: arrayDataTipoDocumento, getDatas: getDatasTipoDocumento } =
    usePetition("cat_tipo_documento/");


const { createFromData } =
    usePetition("ficha_tecnica/");
const props = defineProps({
    idRow: {
        type: String,
    },
    data: {
        type: Object,
    },
});

const viewName = "Ficha Tecnica";
const readOnlyView = ref(false);
const fileName = ref("");

/* const { createData, updateData } = usePetition("ficha_tecnica/"); */

var data: IFichaTecnica = reactive(defaultValues);
const { formState, isValid, errors, showErrors } = useForm(
    data,
    fichaTecnicaValidations
);
const itemId = ref("");

const handleFileUpload = (event: Event) => {
    const target = event.target as HTMLInputElement;
    if (target.files && target.files.length) {
        data.documento = target.files[0];
    }
};

const handleSubmit = async () => {
    data.cartera_proyecto_inversion = props.idRow!!;
    if (isValid.value) {
        const formData = new FormData();
        // Agregar campos del formulario
        formData.append("clave_compromiso", formState.clave_compromiso);
        formData.append(
            "requerimiento_de_hacienda",
            formState.requerimiento_de_hacienda
        );
        formData.append("aforo", formState.aforo);
        formData.append("situacion_actual", formState.situacion_actual);
        formData.append(
            "autorizacion_y_permisos",
            formState.autorizacion_y_permisos
        );
        formData.append(
            "alcance_del_proyecto ",
            formState.alcance_del_proyecto
        );
        formData.append(
            "cartera_proyecto_inversion  ",
            formState.cartera_proyecto_inversion
        );
        formData.append("tipo_obra   ", formState.tipo_obra);
        formData.append("tipo_documento    ", formState.tipo_documento);
        if (data.documento instanceof File) {
            formData.append("documento", data.documento);
        }

        if (fileName.value !== "") {
            delete formState.documento;
        }
        if (itemId.value) {
            /* await updateFromData(formData, itemId.value); */
        } else {
            console.log('entro en else itemid.value: ', formState);
            await createFromData(formData)
                .then(() => {
                    resetForm()
                   // window.location.reload()
                });
            /* await createFromData(formState.value).then(() => window.location.reload()); */
        }
    } else {
        showErrors();
    }
};

const handleCancel = () => window.location.reload();

const updateFormData = () => {
    resetForm();
    readOnlyView.value = false;
    if (props.data && !props.data.loading) {
        data.aforo = props.data.aforo;
        data.alcance_del_proyecto = props.data.alcance_del_proyecto;
        data.autorizacion_y_permisos = props.data.autorizacion_y_permisos;
        data.clave_compromiso = props.data.clave_compromiso;
        data.requerimiento_de_hacienda = props.data.requerimiento_de_hacienda;
        data.situacion_actual = props.data.situacion_actual;
        data.tipo_documento = props.data.tipo_documento;
        data.tipo_obra = props.data.tipo_obra;
        fileName.value = props.data.documento.split("/");
        itemId.value = props.data.cartera_proyecto_inversion;
        readOnlyView.value = true;
        formState.value = props.data
    }
};

const resetForm = () => {
    data.aforo = null;
    data.alcance_del_proyecto = undefined;
    data.autorizacion_y_permisos = undefined;
    data.clave_compromiso = null;
    data.requerimiento_de_hacienda = null;
    data.situacion_actual = undefined;
    data.tipo_documento = null;
    data.tipo_obra = null;
    fileName.value = ''
    data.documento = null
};

watch(
    () => props.data,
    () => {
        updateFormData();
        getDatasTipoObra({ page: 1, size: 100 });
        getDatasTipoDocumento({ page: 1, size: 100 });
    },
    {
        deep: true,
    }
);


</script>
