<template>
  <div class="container-fluid">
    <div class="card-body">
    <div class="table-responsive">
        <div id="dataTable_wrapper" class="dataTables_wrapper dt-bootstrap4">
        <div class="row">
            <div class="col-sm-12 col-md-6">
            </div>
            <div class="col-sm-12 col-md-6">
            <!-- <div id="dataTable_filter" class="dataTables_filter text-right"><label>Tìm kiếm theo biển số:<input
                v-model="licensePlate"
                type="search"
                class="form-control form-control-sm"
                placeholder="Nhập biến số"
                aria-controls="dataTable"></label>
            </div> -->
            </div>
        </div>
        <div class="row">
            <div class="col-sm-12">
            <table class="table table-bordered" id="dataTable">
                <thead>
                <tr>
                <th>ID</th>
                <th>Biển số xe</th>
                <th>Loại xe</th>
                <th>Ngày hết hạn</th>
                </tr>
                </thead>
                <tfoot>
                <tr>
                <th>ID</th>
                <th>Biển số xe</th>
                <th>Loại xe</th>
                <th>Ngày hết hạn</th>
                </tr>
                </tfoot>
                <tbody>
                <tr v-for="(ticket, index) in tickets" :key='index'>
                <td>{{ ticket.IDs }}</td>
                <td>{{ ticket.license_plate }}</td>
                <td>{{ ticket.vehicle_type }}</td>
                <td>{{new Date(ticket.due_date).toISOString().split('T')[0]}}</td>
                </tr>
                </tbody>
            </table>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-12 col-md-5">
            <!-- <div class="dataTables_info" id="dataTable_info" role="status" aria-live="polite">Showing
                {{ (ticketIndex - 1) * 10 + 1 }} to {{ ticketEnd }} of
                {{ vehicleNumber }} tickets
            </div> -->
            </div>
            <div class="col-sm-12 col-md-7">
            <div class="dataTables_paginate paging_simple_numbers" id="dataTable_paginate">
                <ul class="pagination">
                <pagination
                    :page-count="pageOfTickets"
                    :click-handler="getTicketsPage"
                    :prev-text="'Prev'"
                    :next-text="'Next'"
                    :container-class="'pagination'"
                    :prev-class="'previous paginate_button page-item page-link'"
                    :prev-link-class="'previous'"
                    :next-class="'next paginate_button page-item page-link'"
                    :next-link-class="'next'"
                    :page-class="'page-item page-link paginate_button'">
                </pagination>
                </ul>
            </div>
            </div>
        </div>
        </div>
    </div>
    </div>
  </div>
</template>

<script>
import '@/assets/styles/sb-admin-2.min.css'
import '@/assets/styles/dataTables.bootstrap4.css'

import {mapState} from "vuex";
import axios from "axios";

export default {
  name: "CurrentTicketTable",
  data: function () {
    return {
      licensePlate: "",
      ticketIndex: Number,
      ticketsPage: []
    }
  },
  props: {
    currentTickets: [],
  },
  computed: {
    tickets: function () {
      this.ticketsPage.forEach(ticket => {
            // eslint-disable-next-line no-constant-condition
            ticket.vehicle_type = ticket.vehicle_type === 'xe may' ? 'Xe máy' : 'Xe đạp'
            // eslint-disable-next-line no-constant-condition
            ticket.ticket_type = ticket.ticket_type === 'ngay' ? 'Vé ngày' : 'Vé tháng'
          }
      )

      return this.ticketsPage.filter(ticket => ticket.ticket_type === "Vé tháng" && (ticket.license_plate.toLowerCase().includes(this.licensePlate.toLowerCase())));
    },
    ...mapState({
      token: (state) => state.account.user.token,
      vehicleNumber: (state) => state.ticket.vehicleNumber
    }),
    pageOfTickets: function () {
      if (this.vehicleNumber % 10 === 0) {
        return this.vehicleNumber / 10
      }
      return (this.vehicleNumber - this.vehicleNumber % 10) / 10 + 1
    },
    ticketEnd: function () {
      if (this.ticketIndex * 10 > this.vehicleNumber) {
        return this.vehicleNumber
      }
      return this.ticketIndex * 10

    }
  },
  methods: {
    async getTicketsPage(page) {
      try {
        const res = await axios({
          method: "GET",
          url: "http://localhost:3000/api/tickets/month_ticket/" + page,
          headers: {
            "Access-Control-Allow-Origin": "*",
            "Content-Type": "application/json",
            "x-auth-token": this.token,
          },
        });
        if (res.data) {
          this.ticketsPage = res.data;
          console.log(1, this.ticketsPage);
          this.ticketIndex = page;
        }
      } catch (err) {
        alert(err.response.data);
      }
    }
  },
  mounted() {
    this.getTicketsPage(1)
  }
}
</script>

<style scoped>

</style>
