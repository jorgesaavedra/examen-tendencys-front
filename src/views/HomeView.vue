<template>
  <main>
    <div class="home">
      <div class="home-info">

        <div class="accordion" id="accordionExample">
          <div class="accordion-item" v-for="orden in ordenes">
            <h2 class="accordion-header" :id="`heading${orden.id}`">
              <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse"
                :data-bs-target="`#collapseOne${orden.id}`" aria-expanded="true" :aria-controls="`collapse${orden.id}`">
                Orden #{{ orden.number }}
              </button>
            </h2>
            <div :id="`collapseOne${orden.id}`" class="accordion-collapse collapse"
              :aria-labelledby="`heading${orden.id}`" data-bs-parent="#accordionExample">
              <div class="accordion-body">
                <div class="row mb-3">
                  <div class="col-12 d-flex justify-content-end">
                    <button type="button" class="btn btn-primary btn-sm" @click="openModal(orden.id)"><i class="bi bi-plus"></i> Producto</button>
                  </div>
                </div>
                <table class="table">
                  <thead>
                    <tr>
                      <th scope="col">SKU</th>
                      <th scope="col">Name</th>
                      <th scope="col">Quantity</th>
                      <th scope="col">Price</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in orden.items">
                      <th scope="row">{{ item.sku }}</th>
                      <td>{{ item.name }}</td>
                      <td>{{ item.quantity }}</td>
                      <td>{{ item.price }}</td>
                    </tr>
                  </tbody>
                </table>
                <div class="row mb-3">
                  <div class="col-12 d-flex justify-content-end">
                    <button type="button" class="btn btn-primary btn-sm" @click="pagar(orden.id)"><i class="bi bi-credit-card"></i> Pagar</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

      </div>

    </div>

    <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true" @hidden.bs.modal="">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">New message</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form>
              <div class="mb-3 row">
                <label for="SKU" class="col-sm-2 col-form-label">SKU</label>
                <div class="col-sm-10">
                  <input type="text" class="form-control" :class="{ 'border-danger': v$.form.sku.$error }" id="SKU" v-model="form.sku">
                  <div v-if="v$.form.sku.$error" class="text-danger">Campo SKU requerido.</div>
                </div>
              </div>
              <div class="mb-3 row">
                <label for="name" class="col-sm-2 col-form-label">Name</label>
                <div class="col-sm-10">
                  <input type="text" class="form-control" :class="{ 'border-danger': v$.form.sku.$error }" id="name" v-model="form.name">
                  <div v-if="v$.form.name.$error" class="text-danger">Campo Name requerido.</div>
                </div>
              </div>
              <div class="mb-3 row">
                <label for="quantity" class="col-sm-2 col-form-label">Quantity</label>
                <div class="col-sm-10">
                  <input type="number" class="form-control" :class="{ 'border-danger': v$.form.sku.$error }" id="quantity" v-model="form.quantity">
                  <div v-if="v$.form.quantity.$error" class="text-danger">Campo Quantity requerido.</div>
                </div>
              </div>
              <div class="mb-3 row">
                <label for="price" class="col-sm-2 col-form-label">Price</label>
                <div class="col-sm-10">
                  <input type="text" class="form-control" :class="{ 'border-danger': v$.form.sku.$error }" id="price" v-model="form.price">
                  <div v-if="v$.form.price.$error" class="text-danger">Campo Price requerido.</div>
                </div>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
            <button type="button" class="btn btn-primary" @click="agregarProducto()">Agregar</button>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import axios from 'axios';
import { Modal } from "bootstrap";
import { useVuelidate } from '@vuelidate/core';
import { required } from '@vuelidate/validators';

export default {
  data() {
    return {
      v$: useVuelidate(),
      modal: null,
      ordenes: [],
      orden: null,
      form: {
        sku: "",
        name: "",
        quantity: "",
        price: ""
      }
    }
  },
  validations () {
    return {
      form: {
        sku: { required },
        name: { required },
        quantity: { required },
        price: { required }
      }
    }
  },
  mounted() {
    this.initPage();
    this.getOrdenes();
  },
  methods: {
    initPage() {
      let _this = this;
      this.modal = new Modal(document.getElementById('exampleModal'), {});
      document.getElementById('exampleModal').addEventListener('hidden.bs.modal', function (event) {
        _this.onModalClose();
      });
    },
    getOrdenes() {
      let token = 'eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJwUGFINU55VXRxTUkzMDZtajdZVHdHV3JIZE81cWxmaCIsImlhdCI6MTYyMDY2Mjk4NjIwM30.lhfzSXW9_TC67SdDKyDbMOYiYsKuSk6bG6XDE1wz2OL4Tq0Og9NbLMhb0LUtmrgzfWiTrqAFfnPldd8QzWvgVQ';
      axios.get('https://eshop-deve.herokuapp.com/api/v2/orders', { headers: { 'Authorization': `Bearer ${token}` } }).then((response) => {
        this.ordenes = response.data.orders;
      });
    },
    openModal(orden){
      this.orden = orden;
      this.modal.show();
    },
    agregarProducto() {
      this.v$.$validate();
      if (!this.v$.$error) {
        let i = this.ordenes.findIndex((e) => e.id === this.orden);
        if (i >= 0) {
          this.ordenes[i].items.push({sku: this.form.sku, name: this.form.name, quantity: this.form.quantity, price: this.form.price});
        }
      }
    },
    pagar(){
      this.$swal({
        icon: 'success',
        title: 'Proceso exitoso',
        showConfirmButton: true,
        confirmButtonColor: '#0092BC',
        confirmButtonText: 'Continuar'
      });
    },
    onModalClose(){
      this.form = {
        sku: "",
        name: "",
        quantity: "",
        price: ""
      };
      this.v$.$reset();
    }
  }
}    
</script>