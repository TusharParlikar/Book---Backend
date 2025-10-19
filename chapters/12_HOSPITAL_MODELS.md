# Chapter 12: Advanced Data Modeling - Hospital Management

## 12.1 Doctor Model

```javascript
const doctorSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  salary: {
    type: Number,
    required: true
  },
  qualification: {
    type: String,
    required: true
  },
  experience: {
    type: Number,
    default: 0
  },
  specialization: String,
  hospital: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Hospital'
  }
}, { timestamps: true });

const Doctor = mongoose.model('Doctor', doctorSchema);
module.exports = Doctor;
```

## 12.2 Patient Model

```javascript
const patientSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  email: String,
  phoneNumber: String,
  address: {
    street: String,
    city: String
  },
  medicalHistory: [String],
  diagnosis: String,
  bloodGroup: {
    type: String,
    enum: ['O+', 'O-', 'A+', 'A-', 'B+', 'B-', 'AB+', 'AB-']
  },
  gender: {
    type: String,
    enum: ['Male', 'Female', 'Other']
  },
  hospital: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Hospital'
  }
}, { timestamps: true });

const Patient = mongoose.model('Patient', patientSchema);
module.exports = Patient;
```

## 12.3 Hospital Model

```javascript
const hospitalSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  address: {
    street: String,
    city: String,
    state: String,
    zipCode: String
  },
  specializations: [String],
  doctors: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Doctor'
  }],
  patients: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Patient'
  }]
}, { timestamps: true });

const Hospital = mongoose.model('Hospital', hospitalSchema);
module.exports = Hospital;
```

---

## 🎯 Next: Chapter 13 - Security and Authentication
