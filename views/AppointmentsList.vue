<template>
  <div>
    <nav class="navbar navbar-light bg-white shadow-sm mb-4">
      <div class="container d-flex justify-content-between align-items-center">
        <span class="fw-bold fs-5">Doctor Appointments</span>
        <router-link to="/" class="btn btn-outline-primary">⇆ Switch Page</router-link>
      </div>
    </nav>

    <div class="container">
      <div class="card shadow-sm">
        <div class="card-header">
          <h5 class="mb-0">Appointments</h5>
        </div>

        <div class="card-body p-0">
          <table class="table table-bordered table-striped mb-0">
            <thead class="table-light">
              <tr>
                <th>Name</th>
                <th>Symptoms</th>
                <th>Time</th>
                <th>Status</th>
                <th>Update</th>
              </tr>
            </thead>

            <tbody>
              <tr v-if="appointments.length === 0">
                <td colspan="5" class="text-center p-3">No appointments found</td>
              </tr>

              <tr v-for="appointment in appointments" :key="appointment.appointmentId">
                <td>{{ appointment.patientName }}</td>
                <td>{{ appointment.symptoms }}</td>
                <td>{{ appointment.slot }}</td>
                <td>{{ appointment.status }}</td>
                <td>
                  <select
                    class="form-select"
                    :value="appointment.status"
                    @change="e => updateStatus(appointment, e.target.value)"
                  >
                    <option>Pending</option>
                    <option>In Progress</option>
                    <option>Completed</option>
                  </select>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
const API_BASE = "https://khhtfxau6k.execute-api.us-east-1.amazonaws.com/prod";

export default {
  name: "AppointmentsList",

  data() {
    return {
      appointments: []
    };
  },

  mounted() {
    this.fetchAppointments();
  },

  methods: {
    async fetchAppointments() {
      try {
        const res = await fetch(`${API_BASE}/appointments`);

        if (!res.ok) {
          throw new Error(`HTTP ${res.status}`);
        }

        const data = await res.json();

        const parsed = typeof data.body === "string"
          ? JSON.parse(data.body)
          : data;

        this.appointments = parsed;

      } catch (err) {
        console.error("Failed to load appointments:", err);
        alert("Failed to load appointments.");
      }
    },

    async updateStatus(appointment, newStatus) {
      try {
        const res = await fetch(`${API_BASE}/appointments/${appointment.appointmentId}`, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({ status: newStatus })
        });

        const rawBody = await res.text();

        if (!res.ok) {
          throw new Error(`HTTP ${res.status}: ${rawBody}`);
        }

        appointment.status = newStatus;
        alert("Status updated!");

      } catch (err) {
        console.error("Failed to update status:", err);
        alert("Update failed.");
      }
    }
  }
};
</script>
