<template>
  <div class="container">
    <div class="row">
      <div class="email-manager col-12 mt-5">
        <div class="row">
          <div class="col-lg-6">
            <div class="available">
              <h2>Available Recipients</h2>
              <div class="search-container">
                <input type="text" class="form-control" v-model="searchInput" @input="filterRecipients" placeholder="Search by company">
              </div>
              <ul class="recipient-list">
                <li v-for="(group, domain) in groupedAvailableRecipients" :key="domain">
                  <span>{{ domain }}</span>
                  <ul v-if="group.length">
                    <li v-for="recipient in group" :key="recipient.email" @click="toggleRecipient(recipient)">
                      <span>{{ recipient.email }}</span>
                    </li>
                  </ul>
                </li>
              </ul>
            </div>
        </div>

        <div class="  col-lg-6">
          <div class="selected">
            <h2>Selected Recipients</h2>
            <ul class="recipient-list">
              <li v-for="(group, domain) in selectedRecipients" :key="domain">
                <span>{{ domain }}</span>
                <ul v-if="group.length">
                  <li v-for="recipient in group" :key="recipient.email">
                    <span>{{ recipient.email }}</span>
                    <span class="remove-icon" @click="toggleRecipient(recipient)">Remove</span>
                  </li>
                </ul>
              </li>
            </ul>
          </div>
        </div>
        <div class=" col-lg-12 mt-5">
          <div class="add-recipients">
  
  
            <h2>Add Recipient</h2>
            <div class="add-recipient">
              <input type="text" v-model="newRecipient" placeholder="Enter email or company">
              <button @click="addEmail">Add</button>
            </div>
          </div>
        </div>
        </div>
      
      </div>
    </div>
  </div>

</template>

<script>
export default {
  data() {
    return {
      availableRecipients: [
        { email: "ann@timescale.com", isSelected: false },
        { email: "bob@timescale.com", isSelected: false },
        { email: "brian@qwerty.com", isSelected: true },
        { email: "james@qwerty.com", isSelected: false },
        { email: "jane@awesome.com", isSelected: false },
        { email: "kate@qwerty.com", isSelected: true },
        { email: "mike@hello.com", isSelected: true }
      ],
      selectedRecipients: {},
      searchInput: "",
      newRecipient: ""
    };
  },
  created() {
    // Initialize selectedRecipients with selected emails grouped by domain
    this.availableRecipients.forEach((recipient) => {
      if (recipient.isSelected) {
        const domain = recipient.email.split('@')[1];
        this.selectedRecipients[domain] = this.selectedRecipients[domain] || [];
        this.selectedRecipients[domain].push(recipient);
      }
    });
  },
  computed: {
    groupedAvailableRecipients() {
      const grouped = {};
      this.availableRecipients.forEach(recipient => {
        if (!recipient.isSelected) {
          const domain = recipient.email.split('@')[1];
          grouped[domain] = grouped[domain] || [];
          // Add recipient to grouped if it's not already there
          if (!grouped[domain].some(r => r.email === recipient.email)) {
            grouped[domain].push(recipient);
          }
        }
      });
      return grouped;
    }
  },
  methods: {
    toggleRecipient(recipient) {
      recipient.isSelected = !recipient.isSelected;
      if (recipient.isSelected) {
        this.addSelectedRecipient(recipient);
      } else {
        this.removeRecipient(recipient);
      }
    },
    addSelectedRecipient(recipient) {
      const domain = recipient.email.split('@')[1];
      this.selectedRecipients[domain] = this.selectedRecipients[domain] || [];
      this.selectedRecipients[domain].push(recipient);
      this.removeFromAvailableRecipients(recipient);
    },
    removeFromAvailableRecipients(recipient) {
      const index = this.availableRecipients.findIndex(r => r.email === recipient.email);
      if (index !== -1) {
        this.availableRecipients.splice(index, 1);
      }
    },
    removeRecipient(recipient) {
      const domain = recipient.email.split('@')[1];
      const index = this.selectedRecipients[domain].indexOf(recipient);
      if (index !== -1) {
        this.selectedRecipients[domain].splice(index, 1);
        if (this.selectedRecipients[domain].length === 0) {
          delete this.selectedRecipients[domain];
        }
        recipient.isSelected = false;
        this.availableRecipients.push(recipient);
      }
    },
    addEmail() {
      const emailPattern = /\S+@\S+\.\S+/;
      const companyPattern = /^[a-zA-Z\s]*$/;
      if (emailPattern.test(this.newRecipient)) {
        const newRecipient = { email: this.newRecipient, isSelected: false };
        this.availableRecipients.push(newRecipient);
        this.newRecipient = "";
      } else if (companyPattern.test(this.newRecipient)) {
        const domain = this.newRecipient.replace(/\s/g, '').toLowerCase();
        const newRecipient = { email: `${this.newRecipient}@${domain}.com`, isSelected: false };
        this.availableRecipients.push(newRecipient);
        this.newRecipient = "";
      } else {
        alert("Please enter a valid email address or company name.");
      }
    },
    filterRecipients() {
      // If search input is empty, reset available recipients to original state
      if (!this.searchInput.trim()) {
        this.availableRecipients = [
          { email: "ann@timescale.com", isSelected: false },
          { email: "bob@timescale.com", isSelected: false },
          { email: "brian@qwerty.com", isSelected: true },
          { email: "james@qwerty.com", isSelected: false },
          { email: "jane@awesome.com", isSelected: false },
          { email: "kate@qwerty.com", isSelected: true },
          { email: "mike@hello.com", isSelected: true }
        ];
        return;
      }

      const searchTerm = this.searchInput.trim().toLowerCase();

      // Filter available recipients based on search input
      this.availableRecipients = this.availableRecipients.filter(recipient => {
        // Check if recipient's email or company contains the search term
        return recipient.email.toLowerCase().includes(searchTerm) ||
          recipient.email.split('@')[1].toLowerCase().includes(searchTerm);
      });
    }

  }
};
</script>

<style scoped>
.email-manager {
  font-family: Arial, sans-serif;
  /* display: flex; */
}

.recipient-list {
  list-style-type: none;
  padding: 0;
}

.recipient-list li {
  margin-bottom: 5px;
}

.recipient-list span {
  cursor: pointer;
}

.recipient-list span:hover {
  background-color: #f0f0f0;
}

.recipient-list .remove-icon {
  color: red;
  cursor: pointer;
  margin-left: 10px;
}

.search-container {
  margin-bottom: 10px;
}

.add-recipient input[type="text"] {
  margin-bottom: 5px;
  padding: 5px;
}

.add-recipient button {
  padding: 5px 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

.add-recipient button:hover {
  background-color: #45a049;
}
.available,.selected,.add-recipients{
    border: 1px solid #d5d5d5;
    border-radius: 5px;
    padding: 25px 30px;
}
.available h2,.selected h2,.add-recipients h2 {
    border-bottom: 1px solid #d5d5d5;
    margin-bottom: 31px;
    line-height: 2;
}
</style>
