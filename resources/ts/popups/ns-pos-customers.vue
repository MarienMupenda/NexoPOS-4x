<template>
    <div class="bg-white shadow-lg rounded w-95vw h-95vh lg:w-3/5-screen flex flex-col overflow-hidden">
        <div class="p-2 flex justify-between items-center border-b border-gray-400">
            <h3 class="font-semibold text-gray-700">{{ __( 'Customers' ) }}</h3>
            <div>
                <ns-close-button @click="$popup.close()"></ns-close-button>
            </div>
        </div>
        <div class="flex-auto flex p-2 bg-gray-200 overflow-y-auto">
            <ns-tabs :active="activeTab" @active="activeTab = $event">
                <ns-tabs-item identifier="create-customers" label="New Customer">
                    <ns-crud-form 
                        @updated="prefillForm( $event )"
                        @save="handleSavedCustomer( $event )"
                        submit-url="/api/nexopos/v4/crud/ns.customers"
                        src="/api/nexopos/v4/crud/ns.customers/form-config">
                        <template v-slot:title>{{ __( 'Customer Name' ) }}</template>
                        <template v-slot:save>{{ __( 'Save Customer' ) }}</template>
                    </ns-crud-form>
                </ns-tabs-item>
                <ns-tabs-item identifier="account-payment" :label="__( 'Customer Account' )" class="flex" style="padding:0!important">
                    <div class="flex-auto w-full flex items-center justify-center flex-col p-4" v-if="customer === null">
                        <i class="lar la-frown text-6xl text-gray-700"></i>
                        <h3 class="font-medium text-2xl text-gray-700">{{ __( 'No Customer Selected' ) }}</h3>
                        <p class="text-gray-600">{{ __( 'In order to see a customer account, you need to select one customer.' ) }}</p>
                        <div class="my-2">
                            <ns-button @click="openCustomerSelection()" type="info">{{ __( 'Select Customer' ) }}</ns-button>
                        </div>
                    </div>
                    <div v-if="customer" class="flex flex-col flex-auto">
                        <div class="flex-auto p-2 flex flex-col">
                            <div class="-mx-4 flex flex-wrap">
                                <div class="px-4 mb-4 w-full">
                                    <h2 class="font-semibold text-gray-700">{{ __( 'Summary For' ) }} : {{ customer.name }}</h2>
                                </div>
                                <div class="px-4 mb-4 w-full md:w-1/3">
                                    <div class="rounded-lg shadow bg-transparent bg-gradient-to-br from-green-400 to-green-700 p-2 flex flex-col text-white">
                                        <h3 class="font-medium text-lg">{{ __( 'Total Purchases' ) }}</h3>
                                        <div class="w-full flex justify-end">
                                            <h2 class="text-2xl font-bold">{{ customer.purchases_amount | currency }}</h2>
                                        </div>
                                    </div>
                                </div>
                                <div class="px-4 mb-4 w-full md:w-1/3">
                                    <div class="rounded-lg shadow bg-transparent bg-gradient-to-br from-red-500 to-red-700 p-2 text-white">
                                        <h3 class="font-medium text-lg">{{ __( 'Total Owed' ) }}</h3>
                                        <div class="w-full flex justify-end">
                                            <h2 class="text-2xl font-bold">{{ customer.owed_amount | currency }}</h2>
                                        </div>
                                    </div>
                                </div>
                                <div class="px-4 mb-4 w-full md:w-1/3">
                                    <div class="rounded-lg shadow bg-transparent bg-gradient-to-br from-blue-500 to-blue-700 p-2 text-white">
                                        <h3 class="font-medium text-lg">{{ __( 'Account Amount' ) }}</h3>
                                        <div class="w-full flex justify-end">
                                            <h2 class="text-2xl font-bold">{{ customer.account_amount | currency }}</h2>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="flex flex-auto flex-col overflow-hidden">
                                <div class="py-2 w-full">
                                    <h2 class="font-semibold text-gray-700">{{ __( 'Last Purchases' ) }}</h2>
                                </div>
                                <div class="flex-auto flex-col flex overflow-hidden">
                                    <div class="flex-auto overflow-y-auto">
                                        <table class="table w-full">
                                            <thead>
                                                <tr class="text-gray-700">
                                                    <th width="150" class="p-2 border border-gray-200 bg-gray-100 font-semibold">{{ __( 'Order' ) }}</th>
                                                    <th class="p-2 border border-gray-200 bg-gray-100 font-semibold">{{ __( 'Total' ) }}</th>
                                                    <th class="p-2 border border-gray-200 bg-gray-100 font-semibold">{{ __( 'Status' ) }}</th>
                                                    <th width="50" class="p-2 border border-gray-200 bg-gray-100 font-semibold">{{ __( 'Options' ) }}</th>
                                                </tr>
                                            </thead>
                                            <tbody class="text-gray-700">
                                                <tr v-if="orders.length === 0">
                                                    <td class="border border-gray-200 p-2 text-center" colspan="4">{{ __( 'No orders...' ) }}</td>
                                                </tr>
                                                <tr v-for="order of orders" :key="order.id">
                                                    <td class="border border-gray-200 p-2 text-center">{{ order.code }}</td>
                                                    <td class="border border-gray-200 p-2 text-center">{{ order.total | currency }}</td>
                                                    <td class="border border-gray-200 p-2 text-right">{{ order.human_status }}</td>
                                                    <td class="border border-gray-200 p-2 text-center">
                                                        <button v-if="allowedForPayment( order )" class="hover:bg-blue-400 hover:border-transparent hover:text-white rounded-full h-8 px-2 flex items-center justify-center border border-gray bg-white">
                                                            <i class="las la-wallet"></i>
                                                            <span class="ml-1">{{ __( 'Payment' ) }}</span>
                                                        </button>
                                                    </td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="p-2 border-t border-gray-400 flex justify-between">
                            <div></div>
                            <div>
                                <ns-button @click="newTransaction( customer )" type="info">{{ __( 'Account Transaction' ) }}</ns-button>
                            </div>
                        </div>
                    </div>
                </ns-tabs-item>
            </ns-tabs>
        </div>
    </div>
</template>
<script>

import closeWithOverlayClicked from "@/libraries/popup-closer";
import { nsHttpClient, nsSnackBar } from '@/bootstrap';
import { Popup } from '@/libraries/popup';
import nsPosCustomerSelectPopupVue from './ns-pos-customer-select-popup.vue';
import nsCustomersTransactionPopupVue from './ns-customers-transaction-popup.vue';
import { __ } from '@/libraries/lang';
export default {
    name: 'ns-pos-customers',
    data() {
        return {
            activeTab: 'create-customers',
            customer: null,
            subscription: null,
            orders: [],
        }
    },  
    mounted() {
        this.closeWithOverlayClicked();

        this.subscription   =   POS.order.subscribe( order => {
            if ( order.customer !== undefined ) {
                this.activeTab  =   'account-payment';
                this.customer   =   order.customer;
                this.loadCustomerOrders( this.customer.id );
            } else {
                if ( this.$popupParams.customer !== undefined ) {
                    this.activeTab  =   'account-payment';
                    this.customer   =   this.$popupParams.customer;
                    this.loadCustomerOrders( this.customer.id );
                } else if ( this.$popupParams.name !== undefined ) {
                    setTimeout( () => {

                    }, 100 );
                }
            }
        });
    },
    methods: {
        __,
        
        closeWithOverlayClicked,
        
        allowedForPayment( order ) {
            return [ 'unpaid', 'partially_paid', 'hold' ].includes( order.payment_status );
        },

        prefillForm( event ) {
            if ( this.$popupParams.name !== undefined ) {
                event.main.value     =   this.$popupParams.name;
            }
        },

        openCustomerSelection() {
            this.$popup.close();
            Popup.show( nsPosCustomerSelectPopupVue );
        },

        loadCustomerOrders( customerID ) {
            nsHttpClient.get( `/api/nexopos/v4/customers/${customerID}/orders` )
                .subscribe( orders => {
                    this.orders     =   orders;
                });
        },

        newTransaction( customer ) {
            const promise   =   new Promise( ( resolve, reject ) => {
                Popup.show( nsCustomersTransactionPopupVue, { customer, resolve, reject });
            });

            /**
             * let's update the customer
             * with upated reference.
             */
            promise.then( result => {
                POS.loadCustomer( customer.id )
                    .subscribe( _customer => {
                        POS.selectCustomer( _customer );
                    });
            })
        },

        handleSavedCustomer( response ) {
            nsSnackBar.success( response.message ).subscribe();
            POS.selectCustomer( response.entry );
            this.$popup.close();
        }
    }
}
</script>