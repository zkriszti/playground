<template>
  <div class="wrapper">
    <h1>Daily schedule planner</h1>
    <div class="input-fields">
      <p v-for="row in inputRows">{{ row }}</p>
    </div>

    <div class="schedule">    
      <div class="schedule-item" v-for="(item, index) in schedule">
        <div class="timeslot-display">{{ `${item.itemStart}:00` }}</div>
        <div class="timeslot-status" :class="item.status ? 'display-open' : 'display-close'">{{ item.status ? 'open' : 'closed' }}</div>
      </div>
    </div>
    <h2>Summary:</h2>   
    <div class="summary">
      <ul>
        <li v-for="split in summary">From {{ split[0].itemStart }} to {{ split[split.length - 1].itemStart + 1 }}: {{ getDisplayStatus(split[0].status) }}</li>
      </ul>
      <p>Total number of open hours: <span>{{ totalOpenHours }}</span></p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StringInputDisplay',
  data () {
    return {
      earliestStartTime: 8,
      inputString: `From 8-12 – open        
        From 12-19 – open
        From 12-14 – closed
        From 19-21 – closed
        From 21-22 – open
        From 22-24 – closed`
    }
  },
  computed: {
    inputRows () {
      return this.inputString.split('\n').map(str => str.trim())
    },

    formattedInput () {
      let result = this.inputRows
        .map(row => [row.match(/\d+/gm), row.match('open')].flat())
        .map(item => {
          item[0] = parseInt(item[0], 10)
          item[1] = parseInt(item[1], 10)
          item[2] = !!item[2]
          return item
        })
      
      if (result[0][0] > 8) {
        // if input doesn't start with earliestStartTime, we need to add it, assuming it's open:
        const end = result[0][0]
        result.unshift([this.earliestStartTime, end, true])
      }
      return result
    },

    totalOpenHours () {
      return this.schedule.filter(item => item.status).length
    },

    schedule () {   
      // an array of objects stating time and status for each item 
      let i;
      let hourlyValue;   
      let alreadyInScheduleIndex;
      let schedule = []

      this.formattedInput.forEach(block => { 
        hourlyValue = block[2] 
        for (i = block[0]; i < block[1] + 1; i++) {          
          alreadyInScheduleIndex = schedule.findIndex(item => item.itemStart === i)
          if (alreadyInScheduleIndex === -1) {
            schedule.push({itemStart: i, status: hourlyValue})
          } else {
            schedule.splice(alreadyInScheduleIndex, 1, {itemStart: i, status: hourlyValue})
          }
        }
      })
      return schedule      
    },

    summary () {
      let orderedScheduleArray = this.schedule.sort((a, b) => {return a.itemStart < b.itemStart ? -1 : 1}).map(item => item.status);
      const blocks = []

      // first get locations in array where there is a status change
      const changeIndexes = []
      let i
      for (i=1; i < orderedScheduleArray.length; i++) {
        if (orderedScheduleArray[i] !== orderedScheduleArray[i-1]) {
          changeIndexes.push(i)
        }		
      }

      // Break the full schedule into blocks, based on changeIndexes.
      // Basically getting consecutively same statuses into separate blocks
      let j
      for (j = 0; j < changeIndexes.length + 1; j++) {
        let temp
        if (j === 0) { 
          temp = orderedScheduleArray.slice(j, changeIndexes[j])
        } else {
          temp = orderedScheduleArray.slice(changeIndexes[j-1], changeIndexes[j])
        }
        blocks.push(temp)
      }

      // we also need the original (flat array) position of items in the blocks
      // we'll display results as first and last hour of each block, plus status of the first block item
      let currentItemPosition = 0
      const displayResult = blocks.map(block => {
        return block.map(blockItem => {
          const newItem = {itemStart: currentItemPosition + this.earliestStartTime, status: blockItem}
          currentItemPosition++
          return newItem
        })
      })      
      return displayResult
    }
  },

  methods: {
    getDisplayStatus (status) {
      if (status) return "open"
      return "closed"
    }
  }
}
</script>

<style scoped>
h1 {
  text-align: center
}

.input-fields {
  background: #f2f2f2;
  padding: 6px 12px;
  margin: 24px 0 0 0
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

@media only screen and (min-width: 860px) {
  .schedule {
    grid-template-columns: revert;
    grid-auto-flow: column;    
  }
}

</style>
