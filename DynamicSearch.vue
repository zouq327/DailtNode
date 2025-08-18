
<template>
  <div class="dynamic-search-container">
    <!-- 动态表单生成区域 -->
    <div class="form-grid">
      <div v-for="field in searchFields" :key="field.name" class="form-item">
        <component
          :is="getComponentType(field)"
          v-model="searchParams[field.name]"
          v-bind="getComponentProps(field)"
          @change="handleSearch"
        />
      </div>
    </div>

    <!-- 查询结果展示 -->
    <div class="result-container">
      <el-table :data="filteredData" border>
        <el-table-column
          v-for="col in resultColumns"
          :key="col.prop"
          v-bind="col"
        />
      </el-table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 动态字段配置
      searchFields: [
        {
          name: 'keyword',
          label: '关键词',
          type: 'input',
          placeholder: '请输入搜索关键词',
          defaultValue: ''
        },
        {
          name: 'status',
          label: '状态',
          type: 'select',
          options: [
            { value: 1, label: '启用' },
            { value: 0, label: '禁用' }
          ],
          defaultValue: null
        },
        {
          name: 'dateRange',
          label: '日期范围',
          type: 'date-picker',
          dateType: 'daterange',
          defaultValue: []
        }
      ],
      // 动态参数对象
      searchParams: {},
      // 原始数据
      rawData: [],
      // 结果列配置
      resultColumns: [
        { prop: 'id', label: 'ID', width: 80 },
        { prop: 'name', label: '名称' },
        { prop: 'status', label: '状态' }
      ]
    }
  },
  computed: {
    filteredData() {
      return this.rawData.filter(item => {
        return this.searchFields.every(field => {
          const paramValue = this.searchParams[field.name]
          if (!paramValue) return true
          
          switch(field.type) {
            case 'input':
              return item[field.name].includes(paramValue)
            case 'select':
              return item[field.name] === paramValue
            case 'date-picker':
              if (!paramValue.length) return true
              const date = new Date(item[field.name])
              return date >= new Date(paramValue[0]) && 
                     date <= new Date(paramValue[1])
            default:
              return true
          }
        })
      })
    }
  },
  created() {
    this.initSearchParams()
    this.loadData()
  },
  methods: {
    initSearchParams() {
      this.searchParams = this.searchFields.reduce((params, field) => {
        params[field.name] = field.defaultValue
        return params
      }, {})
    },
    getComponentType(field) {
      const componentMap = {
        input: 'el-input',
        select: 'el-select',
        'date-picker': 'el-date-picker'
      }
      return componentMap[field.type] || 'el-input'
    },
    getComponentProps(field) {
      const baseProps = {
        placeholder: field.placeholder || `请选择${field.label}`,
        clearable: true
      }
      
      if (field.type === 'select') {
        return {
          ...baseProps,
          options: field.options
        }
      }
      
      if (field.type === 'date-picker') {
        return {
          ...baseProps,
          type: field.dateType || 'date',
          'value-format': 'yyyy-MM-dd'
        }
      }
      
      return baseProps
    },
    async loadData() {
      this.rawData = await this.$api.getListData()
    },
    handleSearch() {
      console.log('当前查询参数:', this.searchParams)
    }
  }
}
</script>

<style scoped>
.dynamic-search-container {
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
}
.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
}
.form-item {
  display: flex;
  flex-direction: column;
}
.result-container {
  margin-top: 20px;
}
</style>
