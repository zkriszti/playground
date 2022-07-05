<template>
  <div class="wrapper">
  <h1>Daily schedule planner</h1>
    <div class="input-fields">
      <label for="startTime">Start time period at: (8-23)</label>
      <select name="startTime" id="startTime" v-model="modifiedQueueStart">
        <option v-for="n in 16" :value="n - 1 + earliestStartTime">{{ n - 1 + earliestStartTime }}</option>
      </select>

      <label for="closeTime">End time period at: (9-24)</label>
      <select name="closeTime" id="closeTime" v-model="modifiedQueueClose">
        <option v-for="n in 16" :value="n + earliestStartTime">{{ n + earliestStartTime }}</option>
      </select>

      <label for="statusValue">During this period, queue will be:</label>
      <select name="statusValue" id="statusValue" v-model="modifiedStatus">
        <option :value="true">open</option>
        <option :value="false">closed</option>
      </select> 
    </div>

    <div class="cross-check">
      <h2>Data you entered:</h2>
      <p>
        <span>start: {{ `${modifiedQueueStart}:00` }}, </span>
        <span>end: {{ `${modifiedQueueClose}:00` }}, </span>
        <span>status: {{ getDisplayStatus(modifiedStatus) }}</span>         
      </p>           
    </div>
    <div class="input-error-placeholder">
      <small class="input-error" v-if="inputError">Please make sure end time is later than start time!</small>
    </div>
    <div class="schedule">    
      <div class="schedule-item" v-for="(item, index) in schedule">
        <div class="timeslot-display">{{ `${index+8}:00` }}</div>
        <div class="timeslot-status" :class="item ? 'display-open' : 'display-close'">{{ item ? 'open' : 'closed' }}</div>
      </div>
    </div>
    <h2>Summary:</h2>   
    <div class="summary">
      <ul>
        <li v-for="split in displaySummary">From {{ split[0].itemStart }} to {{ split[split.length - 1].itemStart + 1 }}: {{ getDisplayStatus(split[0].status) }}</li>
      </ul>
      <p>Total number of open hours: <span>{{ totalOpenHours }}</span></p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FullyReactiveDisplay',
  data () {
    return {
      schedule: Array(16).fill(true, 0, 16),
      modifiedQueueStart: 8,
      modifiedQueueClose: 24,
      modifiedStatus: true,
      splitSummary: [],
      earliestStartTime: 8
    }
  },
  computed: {
    displaySummary () {
      // we also need the original (flat array) position of items in the splits
      this.currentItemPosition = 0
      const result = this.splitSummary.map(split => {
        return split.map(splitItem => {
          const newItem = {itemStart: this.currentItemPosition + this.earliestStartTime, status: splitItem}
          this.currentItemPosition++
          return newItem
        })
      })
      console.log(result)
      return result
    },

    totalOpenHours () {
      return this.displaySummary.flat().filter(item => item.status).length
    },

    inputError () {
      if (this.modifiedQueueClose <= this.modifiedQueueStart) return true
      return false
    }
  },

  methods: {
    changeSchedule (startTime, closeTime, status) {
      this.splitSummary = []
      let i;
      for (i = startTime - this.earliestStartTime; i < closeTime - this.earliestStartTime; i++) {        
        this.schedule.splice(i, 1, status)
      }
      this.updateSummary()
    },

    getDisplayStatus (status) {
      if (status) return "open"
      return "closed"
    },

    updateSummary () {
      // first get locations in array where there is a status change
      const changeIndexes = []
      let i
      for (i=1; i < this.schedule.length; i++) {
        if (this.schedule[i] !== this.schedule[i-1]) {
          changeIndexes.push(i)
        }		
      }

      // make the full schedule into splits, based on changeIndexes:
      let j
      for (j = 0; j < changeIndexes.length + 1; j++) {
        let temp
        if (j === 0) { 
          temp = this.schedule.slice(j, changeIndexes[j])
        } else {
          temp = this.schedule.slice(changeIndexes[j-1], changeIndexes[j])
        }
        this.splitSummary.push(temp)
      }
    }
  },

  watch: {
    modifiedQueueStart (newVal, oldVal) {
      if (!this.inputError) this.changeSchedule(newVal, this.modifiedQueueClose, this.modifiedStatus)
    },
    modifiedQueueClose (newVal, oldVal) {
      if (!this.inputError) this.changeSchedule(this.modifiedQueueStart, newVal, this.modifiedStatus)
    },
    modifiedStatus (newVal, oldVal) {
      this.changeSchedule(this.modifiedQueueStart, this.modifiedQueueClose, newVal)
    }
  }
}
</script>

<style scoped>
h1 {
  text-align: center
}

.cross-check {
  background: #f2f2f2;
  padding: 6px 12px;
  margin: 24px 0 0 0
}

.input-error-placeholder {
  height: 36px;
  display: flex;
  align-items: center
}

.input-error {
  color: coral
}

.schedule {
  margin-top: 36px;
  display: grid;
  grid-template-columns: repeat(4, min-content);
  grid-row-gap: 18px;
  grid-column-gap: 6px

}

.timeslot-status {
  color: #fefefe
}

.display-open {
  background-color: green
}

.display-close {
  background-color: red
}

.input-fields {
  display: inline-grid;
  grid-auto-flow: row;
  grid-template-columns: minmax(0,1fr) 80px;
  grid-column-gap: 30px;
  grid-row-gap: 12px;
}

@media only screen and (min-width: 860px) {
  .schedule {
    grid-template-columns: revert;
    grid-auto-flow: column;    
  }
}

</style>
