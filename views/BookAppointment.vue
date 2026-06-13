<template>
  <div>
    <nav class="navbar navbar-light bg-white shadow-sm mb-4">
      <div class="container d-flex justify-content-between align-items-center">
        <span class="fw-bold fs-5">Doctor Appointments</span>
        <router-link to="/appointments" class="btn btn-outline-primary">⇆ Switch Page</router-link>
      </div>
    </nav>

    <div class="d-flex justify-content-center align-items-center" style="min-height: 80vh;">
      <div class="card shadow p-5" style="width: 100%; max-width: 700px;">
        <h2 class="text-center mb-4 text-primary">Book an Appointment</h2>
        <form class="row g-3" @submit.prevent="submitAppointment">
          <div class="col-12">
            <input v-model="name" type="text" class="form-control" placeholder="Your Name" required />
          </div>
          <div class="col-12">
            <input v-model="symptoms" type="text" class="form-control" placeholder="Symptoms" required />
          </div>
          <div class="col-12">
            <select v-model="selectedSlot" class="form-select" required>
              <option disabled value="">Select a Time Slot</option>
              <option v-for="slot in slots" :key="slot" :value="slot">
                {{ slot }}
              </option>
            </select>
          </div>
          <div class="col-12">
            <button type="submit" class="btn btn-primary w-100">Book</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BookAppointment",
  data() {
    return {
      name: "",
      symptoms: "",
      selectedSlot: "",
      slots: []
    };
  },
  mounted() {
    fetch("https://khhtfxau6k.execute-api.us-east-1.amazonaws.com/prod/slots")
      .then(res => {
        if (!res.ok) throw new Error(`HTTP Error ${res.status}`);
        return res.json();
      })
      .then(data => {
        console.log("Raw slots response:", data);
        
        // Handle variations in how Lambda Proxy integration formats the payload
        let parsedSlots = [];
        if (data.body) {
          parsedSlots = typeof data.body === "string" ? JSON.parse(data.body) : data.body;
        } else if (Array.isArray(data)) {
          parsedSlots = data;
        }

        // Safely extract names of unbooked slots
        this.slots = parsedSlots.filter(s => !s.isBooked).map(s => s.slot);
      })
      .catch(err => {
        console.error("Error fetching available slots:", err);
        alert("Failed to load available time slots. See console for metadata details.");
      });
  },
  methods: {
    submitAppointment() {
      // Create a clean flat object payload
      const payload = {
        patientName: this.name,
        symptoms: this.symptoms,
        slot: this.selectedSlot
      };

      // Fixed: Send raw flat stringified JSON payload without nested wrappers
      fetch("https://khhtfxau6k.execute-api.us-east-1.amazonaws.com/prod/appointments", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      })
        .then(async res => {
          const text = await res.text();
          if (!res.ok) throw new Error(`HTTP ${res.status}: ${text}`);
          return text ? JSON.parse(text) : {};
        })
        .then(() => {
          alert("Appointment booked successfully!");
          this.name = "";
          this.symptoms = "";
          this.selectedSlot = "";
          
          // Instantly refresh available slots layout locally
          return fetch("https://khhtfxau6k.execute-api.us-east-1.amazonaws.com/prod/slots");
        })
        .then(res => res ? res.json() : null)
        .then(data => {
          if (!data) return;
          let parsedSlots = [];
          if (data.body) {
            parsedSlots = typeof data.body === "string" ? JSON.parse(data.body) : data.body;
          } else if (Array.isArray(data)) {
            parsedSlots = data;
          }
          this.slots = parsedSlots.filter(s => !s.isBooked).map(s => s.slot);
        })
        .catch(err => {
          console.error("Error executing booking routing:", err);
          alert("Failed to complete appointment booking.");
        });
    }
  }
};
</script>
