<template>
		<!-- VENTA POR MARCA Y CATEGORIA -->
	<div>
		<div class="card shadow border-bottom-primary" >
		  	<div class="card-header">TOP 10 de productos</div>
		  	
			<div class="card-body">
			  	<div class="form-row">
			  		<div class="col-md-4 mb-3">
			  			<label for="validationTooltip01">Seleccione tipo de reporte</label>
						<select class="custom-select custom-select-sm" v-bind:class="{ 'is-invalid': validarReporte }" v-model="selectedReporte">
							 <option value="null" selected>Seleccionar</option>
							 <option value=1 selected>Por cantidad vendida</option>
							 <option value=2 selected>Por cantidad ganada</option>
						</select>
						<div class="invalid-feedback">
					        {{messageInvalidReporte}}
					    </div>
			  			
			  			<label for="validationTooltip01">Seleccione Sucursal</label>
						<select class="custom-select custom-select-sm" v-bind:class="{ 'is-invalid': validarSucursal }" v-model="selectedSucursal">
							 <option value="null" selected>Seleccionar</option>
							 <option v-for="sucursal in sucursales" :value="sucursal.CODIGO">{{ sucursal.DESCRIPCION }}</option>
						</select>
						<div class="invalid-feedback">
					        {{messageInvalidSucursal}}
					    </div>

					  	<label class="mt-3" for="validationTooltip01">Seleccione Intervalo de Tiempo</label>
						<div id="sandbox-container">
							<div class="input-daterange input-group">
								   <input type="text" class="input-sm form-control form-control-sm" id="selectedInicialFecha" v-model="selectedInicialFecha" v-bind:class="{ 'is-invalid': validarInicialFecha }"/>
								   <div class="input-group-append form-control-sm">
								   		<span class="input-group-text">a</span>
								   </div>
								   <input type="text" class="input-sm form-control form-control-sm" name="end" id="selectedFinalFecha" v-model="selectedFinalFecha" v-bind:class="{ 'is-invalid': validarFinalFecha }"/>
							</div>
							<div class="invalid-feedback">
					        	{{messageInvalidFecha}}
					    	</div>
						</div>					  

					</div>


					<div class="col-md-4">
						<label for="validationTooltip01">Seleccione Marcas</label> 
						<select multiple class="form-control" size="4" v-model="selectedMarca" :disabled="onMarca" v-bind:class="{ 'is-invalid': validarMarca }">
						   <option v-for="marca in marcas" :value="marca.CODIGO">{{ marca.DESCRIPCION }}</option>
						</select>
						<div class="invalid-feedback">
					        {{messageInvalidMarca}}
					    </div>
						<div class="custom-control custom-switch mt-3">
						  <input type="checkbox" class="custom-control-input" id="customSwitch1" v-on:click="todasMarcas">
						  <label class="custom-control-label" for="customSwitch1" >Seleccionar todas las Marcas</label>
						</div>
					</div>

					
					

				</div>
				<button class="btn btn-dark btn-sm" type="submit" v-on:click="descargar()"><font-awesome-icon icon="download" /> Descargar</button>
			    <button class="btn btn-primary btn-sm" type="submit" v-on:click="llamarDatos">Generar</button>
			</div>
		</div>


		<!-- CARD PARA MARCA Y SU CATEGORIA -->

		<div class="row">

			<!-- SPINNER DESCARGA -->

			<div class="col-md-12">
				<div v-if="descarga" class="d-flex justify-content-center mt-3">
					<strong>Descargando...   </strong>
	                <div class="spinner-border text-success" role="status" aria-hidden="true"></div>
	             </div>
            </div>

			<!-- SPINNER CONSULTA -->

			<div class="col-md-12">
				<div v-if="cargado" class="d-flex justify-content-center mt-3">
					<strong>Cargando...   </strong>
	                <div class="spinner-grow" role="status" aria-hidden="true"></div>
	             </div>
            </div>
            
            <!-- CHART MARCAS -->

            <div class="col-xl-6 col-lg-6">
	                <div class="card-body">
						<div class="ct-chart">
							<canvas id="producto">
								
							</canvas>
						</div>
					</div>
	    	</div>
	     	
	     	<div class="col-xl-6 col-lg-6 mt-3">
	     		<table class="table table-striped table-hover table-light table-sm" v-if="responseVenta.length > 0">
				  <thead>
				    <tr>
				      <th scope="col">#</th>
				      <th scope="col">Codigo</th>
				      <th scope="col">Descripcion</th>
                       <th scope="col">Vendido</th>
				      <th scope="col">Totales</th>
				    </tr>
				  </thead>
				  <tbody>
				    <tr v-for="(producto, index) in responseVenta" v-on:click="clicked(producto)"  data-toggle="modal" data-target="#exampleModalCenter">
				      <th scope="row">{{index+1}}</th>
				      <td>{{producto.COD_PROD}}</td>
				      <td>{{producto.DESCRIPCION}}</td>
				       <td>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(producto.VENDIDO)}}</td>
				      <td>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(producto.PRECIO)}}</td>
				    </tr>
				  </tbody>
				  <tfoot>
					<tr>
					  <th></th>
					  <th>TOTALES</th>
					   <th></th>
					  <th>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(responseVenta.reduce((acc, item) => acc + item.VENDIDO, 0))}}</th>
					  <th>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(responseVenta.reduce((acc, item) => acc + item.PRECIO, 0))}}</th>
					</tr>
				  </tfoot>
				</table>
	     	</div>
	     	<!-- CARD PARA Producto Y SU CATEGORIA -->

			<!-- <div class="card border-left-primary mt-3 col-md-12" v-for="marca in responseMarca">
				<div class="row">
					
					<div class="col-md-6">
						  <div class="card-header font-weight-bold text-primary">
						    {{marca.MARCA}}
						  </div>
				    </div>
				    <div class="col-md-6">
						  <div class="card-header font-weight-bold text-primary text-right">
						    {{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(marca.TOTAL)}}
						  </div>
				    </div>
				</div>  
				
				<ul class="list-group list-group-flush">
				    <li class="list-group-item" v-for="marca in filterItems(responseCategoria, marca.CODIGO)">
				    	<div class="row">
				    		<div class="col-md-6">
				    			{{marca.LINEA}}
				    		</div>
				    		<div class="col-md-6 text-right">
				    			{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(marca.TOTAL)}}
				    		</div>
				    	</div>
					</li>
				</ul>
			</div> -->

			<!-- MODAL DE TABLA PARA DATOS CRUDOS -->

				<div class="modal fade" id="exampleModalCenter" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
				  <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable modal-xl" role="document">
				    <div class="modal-content">
				      <div class="modal-header">
				        <h5 class="modal-title" id="exampleModalCenterTitle">Marca: <small>{{marcaTitulo}}</small></h5>
				        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
				          <span aria-hidden="true">&times;</span>
				        </button>
				      </div>
				      <div class="modal-body">
				        <table class="table" v-if="datosFilas !== null">
						  <thead>
						    <tr>
						      <th scope="col">#</th>
						      <th scope="col">CODIGO</th>
						      <th scope="col">DESCRIPCION</th>
						      <th scope="col">VENDIDO</th>
						      <th scope="col">PRECIO</th>
						    </tr>
						  </thead>
						  <tbody>
						    <tr v-for="(ventas, index) in filterItems(responseVenta, datosFilas)">
						      <th scope="row">{{index+1}}</th>
						      <td>{{ventas.COD_PROD}}</td>
						      <td>{{ventas.DESCRIPCION}}</td>
						      <td>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(ventas.VENDIDO)}}</td>
						      <td>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(ventas.PRECIO)}}</td>
						    </tr>
						  </tbody>
						</table>
				      </div>
				      <div class="modal-footer">
				        <button type="button" class="btn btn-secondary" data-dismiss="modal">Cerrar</button>
				      </div>
				    </div>
				  </div>
				</div>	

			<!-- FIN DE TALA DE DATOS CRUDOS -->

<!-- 			<div class="col-md-12">
				<div class="card border-left-primary mt-3" v-for="marca in responseMarca">
					<div class="row">
						
						<div class="col-md-6">
							  <div class="card-header font-weight-bold text-primary">
							    {{marca.MARCA}}
							  </div>
					    </div>
					    <div class="col-md-6">
							  <div class="card-header font-weight-bold text-primary text-right">
							    {{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(marca.TOTAL)}}
							  </div>
					    </div>
					</div>  
					
					<div class="card-body">
						<table class="table table-sm">
						  <thead class="thead-light">
						    <tr>
						      <th scope="col">#</th>
						      <th scope="col">Categoria</th>
						      <th scope="col">Vendido</th>
						      <th scope="col">Total</th>
						    </tr>
						  </thead>
						  <tbody>
						  <tr v-for="(categoria, index) in filterItems(responseCategoria, marca.CODIGO)">
						      <th scope="row">{{index+1}}</th>
						      <td>{{categoria.LINEA}}</td>
						      <td>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(categoria.VENDIDO)}}</td>
						      <td>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(categoria.TOTAL)}}</td>
						    </tr> 
						  </tbody>
						  <tfoot>
							<tr>
							  <th></th>
							  <th>TOTALES</th>
							  <th>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(marca.VENDIDO)}}</th>
							  <th>{{new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(marca.TOTAL)}}</th>
							</tr>
						  </tfoot>
						</table>
					</div>
				</div>
			</div> -->

		</div>

		<!-- CARD PARA MARCA Y SU CATEGORIA -->

	</div>
		<!-- FIN DE VENTA POR MARCA Y CATEGORIA -->


</template>

<script >
	export default {
        data(){
            return {
              	sucursales: [],
              	selectedSucursal: '',
              	selectedReporte: '',
              	marcas: [],
              	Reporte	:'',
              	selectedMarca: [],
              	datosFilas: null,
              	marcaTitulo: '',
              	/*categorias: [],
              	selectedCategoria: [],*/
              	onMarca: false,
              /*	onCategoria: false,*/
              	validarSucursal: false,
              		validarReporte: false,
              	messageInvalidSucursal: '',
              		messageInvalidReporte: '',
              	validarMarca: false,
              	messageInvalidMarca: '',
              	/*validarCategoria: false,
              	messageInvalidCategoria: '',*/
              	selectedInicialFecha: '',
              	validarInicialFecha: false,
              	messageInvalidFecha: '',
              	selectedFinalFecha: '',
              	validarFinalFecha: false,
              	datos: {},
              	responseMarca: {},
              	responseProducto: {},
            /*  	responseCategoria: [],*/
              	responseVenta: [],
              	varTotalProducto: [],
				varNombreProducto: [],
              	cargado: false,
              	descarga: false
            }
        }, 
        methods: {
            llamarBusquedas(){	
	          axios.get('busquedas/').then((response) => {
	           	this.sucursales = response.data.sucursales;
	           	this.marcas = response.data.marcas;
	           	/*this.categorias = response.data.vendedor;*/
	          }); 
	        },
	        descargar(){
	        	let me = this;	
	        	if(this.generarConsulta() === true) {
	        		me.descarga = true;
		        	axios({
					  url: '/export',
					  method: 'POST',
					  data: me.datos,
					  responseType: 'blob', // important
					}).then((response) => {
						me.descarga = false;
					   const url = window.URL.createObjectURL(new Blob([response.data]));
					   const link = document.createElement('a');
					   link.href = url;
					   link.setAttribute('download', 'Venta-'+me.selectedInicialFecha+'-'+me.selectedFinalFecha+'.xlsx'); //or any other extension
					   document.body.appendChild(link);
					   link.click();
					});
				}
	        },
	        clicked(row) {
	       	  this.marcaTitulo = row.MARCA; 	
		      this.datosFilas = row.CODIGO;
		    },
	        filterItems: function(items, codigo) {
			      return items.filter(function(item) {
			      return item.MARCA === codigo;
			    })
			 },
	        todasMarcas(e){
	        	this.onMarca = !this.onMarca;
	        },
/*	        todasCategorias(e){
	        	this.onCategoria = !this.onCategoria;
	        },*/
	        llamarDatos(){
	        	let me = this;	
	        	if(this.generarConsulta() === true && this.selectedReporte==1) {
	        		me.cargado = true;
					axios.post('/CantidadArticulos', this.datos).then(function (response) {
						me.cargado = false;
						me.responseVenta = response.data.ventas;
						
					    const VentasArray = Object.keys(response.data.ventas).map(i => response.data.ventas[i])
					    me.responseProducto = VentasArray
					   /* const categoriaArray = Object.keys(response.data.categorias).map(i => response.data.categorias[i])
					    me.responseCategoria = categoriaArray*/
					    me.loadProductos();
					});
	        	} else {
	        		if(this.generarConsulta() === true && this.selectedReporte==2) {
                    me.cargado = true;
					axios.post('/MontoArticulos', this.datos).then(function (response) {
						me.cargado = false;
						me.responseVenta = response.data.ventas;
						
					    const VentasArray = Object.keys(response.data.ventas).map(i => response.data.ventas[i])
					    me.responseProducto = VentasArray
					   /* const categoriaArray = Object.keys(response.data.categorias).map(i => response.data.categorias[i])
					    me.responseCategoria = categoriaArray*/
					    me.loadProductos();
					});
	        			}else{
	        		alert("false");
	        		}
	        	}
	        },
	        generarConsulta(){
	        	
	        	
	        	if (this.selectedSucursal === null || this.selectedSucursal === "null") {
	        		this.validarSucursal = true;
	        		this.messageInvalidSucursal = 'Por favor seleccione sucursal';
	        		return false;
	        	} else {
	        		this.validarSucursal = false;
	        		this.messageInvalidSucursal = '';
	        	}	
	        	  	if (this.selectedReporte === null || this.selectedReporte === "null") {
	        		this.validarReporte = true;
	        		this.messageInvalidReporte = 'Por favor seleccione tipo de Reporte';
	        		return false;
	        	} else {
	        		this.validarReporte = false;
	        		this.messageInvalidReporte = '';
	        	}	

	        	if(this.onMarca === false && this.selectedMarca === null) {
	        		this.validarMarca = true;
	        		this.messageInvalidMarca = 'Por favor seleccione una o varias Marcas';
	        		return false;
	        	} else {
	        		this.validarMarca = false;
	        		this.messageInvalidMarca = '';
	        	}

	        	/*if(this.onCategoria === false && this.selectedCategoria === null) {
	        		this.validarCategoria = true;
	        		this.messageInvalidCategoria = 'Por favor seleccione una o varias Categorias';
	        		return false;
	        	} else {
	        		this.validarCategoria = false;
	        		this.messageInvalidCategoria = '';
	        	}*/

	        	if(this.selectedInicialFecha === null || this.selectedInicialFecha === "") {
	        		this.validarInicialFecha = true;
	        		this.messageInvalidFecha = 'Por favor seleccione una fecha Inicial';
	        		return false;
	        	} else {
	        		this.validarInicialFecha = false;
	        		this.messageInvalidFecha = '';
	        	}

	        	if(this.selectedFinalFecha === null || this.selectedFinalFecha === "") {
	        		this.validarFinalFecha = true;
	        		this.messageInvalidFecha = 'Por favor seleccione una fecha Final';
	        		return false;
	        	} else {
	        		this.validarFinalFecha = false;
	        		this.messageInvalidFecha = '';
	        	}		

	        	if(this.onMarca === true) {
	        		for (var key in this.marcas){
	        			this.selectedMarca[key] = this.marcas[key].CODIGO;
	        		}
	        	} 

	        	/*if(this.onCategoria === true) {
	        		for (var key in this.categorias){
	        			this.selectedCategoria[key] = this.categorias[key].CODIGO;
	        		}
	        	}*/

	        	this.datos = {
	        	Sucursal: this.selectedSucursal,
	        	Inicio: String(this.selectedInicialFecha),
	        	Final: String(this.selectedFinalFecha),
	        	Marcas: this.selectedMarca,
	        	Reporte: this.selectedReporte,	
	        /*	Categorias: this.selectedCategoria,*/
	        	AllBrand: this.onMarca,
	        	/*AllCategory: this.onCategoria */
	        	};
	        	
	        	return true;
	        },
	        loadProductos(){
				let me = this;
				me.varNombreProducto = [];
				me.varTotalProducto = [];
				me.responseProducto.map(function(x){
					me.varNombreProducto.push(x.COD_PROD);
					me.varTotalProducto.push(x.PRECIO);
				});

				me.varProducto = document.getElementById('producto').getContext('2d');

				 me.charProducto = new Chart(me.varProducto, {
				    type: 'bar',
				    data: {
				        labels: me.varNombreProducto,
				        datasets: [{
				            label: 'Productos',
				            data: me.varTotalProducto,
				            backgroundColor: 'rgba(75, 192, 192, 0.2)',
				            borderColor: 'rgba(75, 192, 192, 1)',
				            borderWidth: 1
				        }]
				    },
				    options: {
				    	tooltips: {
				              callbacks: {
				                  label: function(tooltipItem, data) {
				                      var value = data.datasets[0].data[tooltipItem.index];
				                      
				                      return 'Gs. ' + new Intl.NumberFormat("de-DE", {style: "decimal", decimal: "0"}).format(value) + '';
				                  }
				              }
				          },
				        scales: {
				            yAxes: [{
				                ticks: {
				                    beginAtZero: true,
				                    callback: function(value, index, values) {
							          return value.toLocaleString();
							        }
				                }
				            }]
				        }
				    }
				});
			}
        },
        mounted() {
        	let me = this;
        	$(function(){
		   		    $('#sandbox-container .input-daterange').datepicker({
		   		    	    keyboardNavigation: false,
    						forceParse: false
    				});
    				$("#selectedInicialFecha").datepicker().on(
			     		"changeDate", () => {me.selectedInicialFecha = $('#selectedInicialFecha').val()}
					);
					$("#selectedFinalFecha").datepicker().on(
			     		"changeDate", () => {me.selectedFinalFecha = $('#selectedFinalFecha').val()}
					);

					$('table').dataTable();
			});
			this.llamarBusquedas();
        }
    }    
</script>
