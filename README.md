let tagconditions = {
  'type': 'tag', // generated by tableid
  'tag': '11111111-2222-3333-4444-123456789012', //persisted
  'conditions': { // generated
     "fact": "tags",
     "operator": "in",
     "value": '11111111-2222-3333-4444-123456789012'
   }
}
let eligconditions =  {
  'type': 'eligibility',
  'equationId': '11111111-2222-3333-4444-5678901234567',
  'eligibilityId': '11111111-2222-3333-4444-123456789012',
  'fieldid': 'age'
}


let JsonRules = require('json-rules')
let achievementRules = JsonRules('Achievements') //shares a universe of rules
// DEFINING RULES
// operators:
// "in", "notIn", "lessThan", "lessThanInclusive", "greaterThan", "greaterThanInclusive", "equal", "notEqual"
// priority: integer, not exclusive; can be multiple 1's
// conditions: object representing
// action: callback to be executed when conditions passes
//         actions can modify facts
achievementRules.addRule({
  id: "age-range"
  priority: 1,
  conditions: {
    all: [{
     "id": "6ed20017-375f-40c9-a1d2-6d7e0f4733c5",

     // option 1
     "fact": {
        name: "eligibility",
        params: {
          equationId: '11111111-2222-3333-4444-5678901234567',
          eligibilityId: '11111111-2222-3333-4444-123456789012',
          fieldid: 'age'
        }
     },
     "operator": "lessThan",
     "value": 45,

     // option 2
      fact: "eligibility",
      params: {
        equationId: '11111111-2222-3333-4444-5678901234567',
        eligibilityId: '11111111-2222-3333-4444-123456789012',
        fieldid: 'age'
      },
     "operator": "lessThan",
     "value": 45
   },
   {
    "id": "0941d27f-8154-439a-a720-ed4b48c196eb",
     "fact": "age",
     "operator": "greaterThan",
     "value": 21
   }]
 },
 action: {
    type: "reward",
    params: {
      reward: {
        type: "currency",
        detail: {
          currency: "points"
          amount: 50
        }
      }
    }
 }
})

achievementRules.addRule({
  all: [{
   "fact": "tags",
   "operator": "in",
   "value": '11111111-2222-3333-4444-123456789012'
 }]
})

// ADDING FACTS
// facts are persisted across each run() of the rules
instance.addFact("age", (params, facts, done) => {
  // api call to load demographic data
  // need access to facts
  request(`/users/${facts('userId')}`)
  done(null, age)
}, { cache: true })

//    addFact OPTIONS
//      cache:  persists across each run of the rules engine, default: true
//              can also receive a callback, which can return true or false to denote caching

// RETRIEVING FACTS
r.getFact(factName, params = {}) //searches the internal facts hash map


// RUNNING RULES
// rules.run(facts)
// all other methods should raise an exception if called after run()
instance.run({
  userId: '0e03b9bd-36f9-4933-9ae8-2165baeaccde',
  performedAt: '2015-05-25',
  tags: ['11111111-2222-3333-4444-123456789012']
})

instance.on('error', (r, rule) {
  // log failure reason
})
instance.on('failure', (r, rule) {
  // log failure reason
})
instance.on('action', (r, action) {
  // log failure reason
})

// HANDLING ACTIONS
// disadvantage is that if the persisted version changes, the code breaks
instance.on('reward', (r, data) => {
  data
  // can this stop the rules?
  // alternatively, could just add some control properties to action
  //  e.g. { emit: reward, stop: true, data: {} }
})

instance.onAction((r, rule, done) {
  let action = rule.action
  switch (action.type) {
    case 'reward':
      let grantAmount = action.data.reward.amount
      if(r.getFact('pointCaps')[action.data.currency] < grantAmount) {
        grantAmount = r.getFacts('pointCaps')[action.data.currency]
      }
      if(grantAmount > 0) {
        grant(grantAmount)
      }
      break;
    case 'pointCapReached':
      // is this the lowest point cap for this currency?
      let newPointCap = Math.max(r.facts.pointBalance - action.data.pointCap)
      r.facts.pointCaps = r.facts.pointCaps || {}
      if(!r.facts.pointCaps[action.data.currency] || r.facts.pointCaps[action.data.currency] < newPointCap) {
        r.facts.pointCaps[action.data.currency] = newPointCap
      }
      break;
    default:
      done()
      break;
  }
})

// when a rule runs, it works from the outside-in
// when it encounters a fact that is not defined, it will emit an error event, and continue to
// the next rule
//
//
// POINT CAPS
//  single user who meets certain criteria, limit to X points
achievementRules.addRule({
  id: "point-cap"
  priority: 100,
  conditions: {
    all: [{
     "fact": "age",
     "operator": "lessThan",
     "value": 45
   },
   {
     "fact": "pointBalance",
     "operator": "greaterThanInclusive",
     "value": 1000
   }]
 },
 action: {
    type: "pointCapReached",
    params: {
      currency: 'points',
      pointCap: 1000
    }
 }
})

//  family user, X points per, Y points per family
achievementRules.addRule({
  id: "point-cap"
  priority: 100,
  conditions: {
    all: [
      {
       "fact": "inFamily",
       "operator": "equal",
       "value": true
     },
     {
       "fact": "familyPointBalance",
       "operator": "greaterThanInclusive",
       "value": 5000
     }
    ]
 },
 action: {
    type: "pointCapReached",
    params: {
      currency: 'points',
      pointCap: 5000
    }
 }
})

// rate limits
achievementRules.addRule({
  id: "point-cap"
  priority: 100,
  conditions: {
    all: [{
     "fact": {
        name: "rateLimit",
        params: {
          per: 30
        }
      }
     "operator": "lessThanInclusive",
     "value": 1
   }]
 },
 action: {}
})

//  date ranges
achievementRules.addRule({
  id: "point-cap"
  priority: 100,
  conditions: {
    all: [
    {
     "fact": {
        name: "dateRange",
        params: {
          startAt: '1994-11-05T13:15:30.000Z',
          endAt: '2014-11-05T13:15:30.000Z'
        }
     },
     "operator": "equal",
     "value": true
   }
   ]
 },
 action: {}
})
// use json-schema for the basic validations?


// hash the fact to cache: https://github.com/puleos/object-hash
// add debug() so can output things like 'loading from cache: fact: XXX, params: YYYY'
