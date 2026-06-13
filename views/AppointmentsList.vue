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
              <tr v-for="appointment in appointments" :key="appointment.appointmentId">
                <td>{{ appointment.patientName }}</td>
                <td>{{ appointment.symptoms }}</td>
                <td>{{ appointment.slot }}</td>
                <td>{{ appointment.status }}</td>
                <td>
                  <select class="form-select" :value="appointment.status" @change="e => updateStatus(appointment, e.target.value)">
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
    fetchAppointments() {
      fetch("https://khhtfxau6k.execute-api.us-east-1.amazonaws.com/prod/appointments")
        .then(res => {
          if (!res.ok) throw new Error(`HTTP Error ${res.status}`);
          return res.json();
        })
        .then(data => {
          console.log("Raw appointments response:", data);
          
          // Safely unpack data array variant structures
          if (data.body) {
            this.appointments = typeof data.body === "string" ? JSON.parse(data.body) : data.body;
          } else if (Array.isArray(data)) {
            this.appointments = data;
          }
        })
        .catch(err => {
          console.error("Error listing admin data collections:", err);
        });
    },
    updateStatus(appointment, newStatus) {
      console.log("Processing update for target ID:", appointment.appointmentId);

      // Construct flat payload containing partition key identifier alongside parameters
      const payload = { 
        appointmentId: appointment.appointmentId,
        status: newStatus 
      };

      // Base URL target mapped dynamically
      const url = `https://khhtfxau6k.execute-api.us-east-1.amazonaws.com/prod/appointments/${appointment.appointmentId}`;

      fetch(url, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(payload) // Directly stringified single payload level
      })
        .then(async res => {
          const rawBody = await res.text();
          if (!res.ok) {
            throw new Error(`HTTP ${res.status}: ${rawBody}`);
          }
          return rawBody ? JSON.parse(rawBody) : {};
        })
        .then(() => {
          alert("Status updated successfully!");
          this.fetchAppointments(); // Re-trigger tracking array sync refresh
        })
        .catch(err => {
          console.error("Failed to execute infrastructure update pipeline:", err);
          alert("Update operational process failed. Review console logs.");
        });
    }
  }
};
</script>
