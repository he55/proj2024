<template>
    <el-card style="width: 100%;height: 100%;box-sizing:border-box;">
        <template #header>
            <el-button type="primary" @click="mode = 1; formVisible = true">添加药品</el-button>
            <el-button type="primary" @click="refreshData">刷新</el-button>
            <input type="file" @change="handleImport">
            <input type="file" @change="handleFileUpload">
        </template>
        <el-table :data="dataList" stripe border style="height: 500px;">
            <el-table-column label="操作" width="140">
                <template #default="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">修改</el-button>
                    <el-button size="small" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
            <el-table-column prop="drugId" label="Id" />
            <el-table-column prop="drugName" label="药品名称" width="150" />
            <el-table-column prop="spec" label="药品规格" width="150" />
            <el-table-column prop="pinYin" label="拼音码"  width="150"/>
            <el-table-column prop="sort" label="排序编号" width="90" />
            <el-table-column prop="clinicalUnit" label="临床单位" width="90" />
            <el-table-column prop="packageUnit" label="包装单位" width="90" />
            <el-table-column prop="c2PQuantity" label="转换比" />
            <el-table-column prop="approvalNumber" label="批准文号" width="200" />
            <!-- <el-table-column prop="drugCode" label="本地药品编码" /> -->
            <el-table-column prop="nationDrugCode" label="国家药品编码" width="120" />
            <el-table-column prop="radManufacturer" label="药品研发厂家" width="200" />
            <el-table-column prop="dosageForm" label="药品剂型" width="90" />
            <el-table-column prop="defaultSaleUnit" label="销售单位" width="90" />
            <el-table-column prop="clinicalSaleUnit" label="临床销售单位" width="120" />
            <el-table-column prop="packagePrice" label="包装单位销售单价" />
            <el-table-column prop="clinicalPrice" label="临床销售单价" />
        </el-table>
        <el-pagination
        style="width: 100%;"
      v-model:current-page="currentPage4"
      v-model:page-size="pageSize4"
      :page-sizes="[100, 200, 300, 400]"
      layout="total, sizes, prev, pager, next, jumper"
      :total="400"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
    </el-card>

    <el-dialog v-model="formVisible" title="添加药品" width="850" :close-on-click-modal="false">
        <el-form :inline="true" ref="formRef" :model="dataModel" :rules="rules" label-width="auto">
            <el-form-item label="药品名称" prop="drugName">
                <el-input v-model="dataModel.drugName" autocomplete="off" />
            </el-form-item>
            <el-form-item label="药品规格" prop="spec">
                <el-input v-model="dataModel.spec" autocomplete="off" />
            </el-form-item>
            <el-form-item label="拼音码" prop="pinYin">
                <el-input v-model="dataModel.pinYin" autocomplete="off" />
            </el-form-item>
            <el-form-item label="排序编号" prop="sort">
                <el-input v-model="dataModel.sort" type="number" autocomplete="off" />
            </el-form-item>
            <el-form-item label="临床单位" prop="clinicalUnit">
                <el-input v-model="dataModel.clinicalUnit" autocomplete="off" />
            </el-form-item>
            <el-form-item label="包装单位" prop="packageUnit">
                <el-input v-model="dataModel.packageUnit" autocomplete="off" />
            </el-form-item>
            <el-form-item label="临床到包装单位的转换" prop="c2PQuantity">
                <el-input v-model="dataModel.c2PQuantity" type="number" autocomplete="off" />
            </el-form-item>
            <el-form-item label="批准文号" prop="approvalNumber">
                <el-input v-model="dataModel.approvalNumber" autocomplete="off" />
            </el-form-item>
            <el-form-item label="本地药品编码" prop="drugCode">
                <el-input v-model="dataModel.drugCode" autocomplete="off" />
            </el-form-item>
            <el-form-item label="国家药品编码" prop="nationDrugCode">
                <el-input v-model="dataModel.nationDrugCode" autocomplete="off" />
            </el-form-item>
            <el-form-item label="药品研发厂家" prop="radManufacturer">
                <el-input v-model="dataModel.radManufacturer" autocomplete="off" />
            </el-form-item>
            <el-form-item label="药品剂型" prop="dosageForm">
                <el-input v-model="dataModel.dosageForm" autocomplete="off" />
            </el-form-item>
            <el-form-item label="销售单位" prop="defaultSaleUnit">
                <el-input v-model="dataModel.defaultSaleUnit" type="number" autocomplete="off" />
            </el-form-item>
            <el-form-item label="临床销售单位" prop="clinicalSaleUnit">
                <el-input v-model="dataModel.clinicalSaleUnit" type="number" autocomplete="off" />
            </el-form-item>
            <el-form-item label="包装单位销售单价" prop="packagePrice">
                <el-input v-model="dataModel.packagePrice" type="number" autocomplete="off" />
            </el-form-item>
            <el-form-item label="临床销售单价" prop="clinicalPrice">
                <el-input v-model="dataModel.clinicalPrice" type="number" autocomplete="off" />
            </el-form-item>
        </el-form>
        <template #footer>
            <div class="dialog-footer">
                <el-button @click="formVisible = false; formRef.resetFields()">取消</el-button>
                <el-button type="primary" @click="handleCreate">确认</el-button>
            </div>
        </template>
    </el-dialog>

</template>

<script setup>
import { addDrugService, batchAddDrugsService, deleteDrugService, getAllDrugsService, getDrugService, updateDrugService, uploadExcelFileService } from '@/api/drug';
import { ElMessage, ElMessageBox } from 'element-plus';
import { pinyin } from 'pinyin-pro';
import readXlsxFile from 'read-excel-file';
import { ref, watch } from 'vue';

const formRef = ref()
const formVisible = ref(false)
const dataModel = ref({
    drugName: '',
    spec: '',
    pinYin: '',
    sort: 0,
    clinicalUnit: '',
    packageUnit: '',
    c2PQuantity: 1,
    approvalNumber: '',
    drugCode: '',
    nationDrugCode: '',
    radManufacturer: '',
    dosageForm: '',
    defaultSaleUnit: 0,
    clinicalSaleUnit: 0,
    packagePrice: 0,
    clinicalPrice: 0
})
const rules = {
    drugName: [{ required: true, message: '必填', trigger: 'blur' }],
    spec: [{ required: true, message: '必填', trigger: 'blur' }],
    pinYin: [{ required: true, message: '必填', trigger: 'blur' }],
    sort: [{ required: true, message: '必填', trigger: 'blur' }],
    clinicalUnit: [{ required: true, message: '必填', trigger: 'blur' }],
    packageUnit: [{ required: true, message: '必填', trigger: 'blur' }],
    c2PQuantity: [{ required: true, message: '必填', trigger: 'blur' }],
    approvalNumber: [{ required: true, message: '必填', trigger: 'blur' }],
    drugCode: [{ required: true, message: '必填', trigger: 'blur' }],
    nationDrugCode: [{ required: true, message: '必填', trigger: 'blur' }],
    radManufacturer: [{ required: true, message: '必填', trigger: 'blur' }],
    dosageForm: [{ required: true, message: '必填', trigger: 'blur' }],
    defaultSaleUnit: [{ required: true, message: '必填', trigger: 'blur' }],
    clinicalSaleUnit: [{ required: true, message: '必填', trigger: 'blur' }],
    packagePrice: [{ required: true, message: '必填', trigger: 'blur' }],
    clinicalPrice: [{ required: true, message: '必填', trigger: 'blur' }],
}

const currentPage4 = ref(4)
const pageSize4 = ref(100)
const handleSizeChange = (val) => {
  console.log(`${val} items per page`)
}
const handleCurrentChange = (val) => {
  console.log(`current page: ${val}`)
}

watch(() => dataModel.value.drugName, (name) => {
    const resu = pinyin(name, { toneType: "none", type: "array" })
    dataModel.value.pinYin = resu.map(v => v.at(0).toUpperCase()).join('')
    console.log(name, resu)
})

const dataList = ref()
const getList = async () => {
    dataList.value = await getAllDrugsService()
}
getList()

const refreshData = async () => {
    await getList()
    ElMessage.success('刷新成功')
}

let mode // 0 查看; 1 创建; 2 修改
const handleCreate = async () => {
    await formRef.value.validate()

    if (mode === 1) {
        await addDrugService(dataModel.value)
        ElMessage.success('药品添加成功')
    } else if (mode === 2) {
        await updateDrugService(dataModel.value.drugId, dataModel.value)
        ElMessage.success('药品修改成功')
    }

    formVisible.value = false
    formRef.value.resetFields()

    await getList()
}

const handleEdit = async (index, row) => {
    dataModel.value = await getDrugService(row.drugId)
    mode = 2
    formVisible.value = true
}

const handleDelete = async (index, row) => {
    await ElMessageBox.confirm(
        '确认删除数据',
        '提示',
        {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning'
        }
    )

    await deleteDrugService(row.drugId)
    await getList()
}

const handleImport = async (e) => {
    if (e.target.files.length) {
        const rows = await readXlsxFile(e.target.files[0])
        const drugs = rows.slice(3, 5).map(item => {
            const drug = {
                drugName: item[2],
                approvalNumber: item[1],
                dosageForm: item[7],
                spec: item[8],
                nationDrugCode: item[9],
                drugId:0,
                clinicalUnit:'无',
                packageUnit:'无',
                c2PQuantity:1,
                drugCode:'无',
                radManufacturer:'',
                packagePrice:1,
                clinicalPrice:1,
            }

            const resu = pinyin(drug.drugName, { toneType: "none", type: "array" })
            drug.pinYin = resu.map(v => v.at(0).toUpperCase()).join('')
            return drug
        })
        await batchAddDrugsService(drugs)
        ElMessage.success('药品导入成功')
        await getList()
    }
}

const handleFileUpload=async(e)=>{
    if(e.target.files.length){
        let file = e.target.files[0]
        await uploadExcelFileService(file)
        ElMessage.success('上传成功')
        await getList()
    }
}
</script>