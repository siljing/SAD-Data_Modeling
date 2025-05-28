# Data Modeling

### **1. 实体定义矩阵（Entity Definition Matrix）**  
| **实体名称**          | **业务定义**                                                                 |
|-----------------------|----------------------------------------------------------------------------|
| **Employee**          | 公司员工的核心信息，包括唯一标识、姓名、联系方式、雇佣状态等。每个员工拥有唯一的EmployeeID，即使离职后ID也不会重复使用。需支持员工自助更新个人信息（如地址、电话），并关联其所属部门、紧急联系人、捐款和储蓄债券记录。                                       | 
| **Department**        | 公司部门信息，记录部门编号、名称及所属站点（如Orlando、Denver）。每个部门属于一个站点，一名员工仅归属于一个部门。部门信息用于组织架构管理和数据权限控制。                                               |
| **EmergencyContact**  | 员工的紧急联系人信息，包括姓名、关系、联系方式（家庭电话、工作电话）。一名员工可登记多个紧急联系人（如配偶、父母）。                                           | 
| **UnitedWayContribution** | 员工对United Way的捐款记录。包括捐款类型（固定比例、一次性金额、自定义金额）、生效日期、扣款频率（每期工资扣除）。每位员工每年需提交一次更新表单。                            | 
| **SavingsBond**       | 员工购买的美国储蓄债券记录。每笔债券需记录面额（$100、$200等）、扣款类型（一次性或定期）、债券所有者信息（员工本人或其他受益人，如子女）。每位员工可为多个受益人购买债券，需填写独立表单。                              |
| **Site**              | 公司站点信息，包括站点名称（如Orlando、Denver）、地址、员工数量。用于管理多站点数据分布，确保电话簿和邮件投递按站点分类。                                    |
| **TelephoneListing**  | 员工电话及办公地点信息，包括分机号、建筑编号、房间号、邮件停靠点（Mail Stop）。与员工实体直接关联，需支持实时更新以替代纸质电话簿。                                        |
| **BondOwner**         | 储蓄债券的所有者信息（可能为非员工，如员工子女）。记录所有者的姓名、SSN、联系地址，确保债券归属清晰。                                   |
| **EmploymentHistory**         | （高级选项）员工的雇佣历史记录，包括入职日期、离职日期、雇佣状态（在职、退休、离职、休假）、雇佣类型（全职/兼职）。支持员工多次入职的场景，保留历史数据以计算福利资格。                                   |
---

### 补充说明  
1. **实体关系**：  
   - **Employee**与**Department**为多对一关系（一个部门包含多名员工）。  
   - **Employee**与**EmergencyContact**为一对多关系（一名员工可有多个紧急联系人）。  
   - **Employee**与**SavingsBond**为一对多关系（一名员工可为多个受益人购买债券）。  
   - **Site**与**Department**为一对多关系（一个站点包含多个部门）。  

2. **关键业务规则**：  
   - **EmployeeID**为不可重复的序列号，即使员工离职也不回收。  
   - **SavingsBond**的所有者（BondOwner）需独立管理，避免与员工信息混淆。  
   - **TelephoneListing**的更新需实时同步至所有关联系统（如工资系统、邮件系统）。  

3. **高级扩展**：  
   - **EmploymentHistory**实体支持员工状态跟踪，通过`EmploymentStatus`字段（枚举值：Active/Retired/Terminated/Leave）和`EmploymentType`字段（Full-Time/Part-Time）实现灵活管理。  
   - 通过`HireDate`和`TerminationDate`记录多次雇佣历史，确保福利计算的准确性（如养老金、医疗保险）。  

---

### **2. 上下文数据模型（Context Data Model）**  
![](https://via.placeholder.com/800x400.png?text=Context+Data+Model+Diagram)  
- **实体关系说明**：  
  - **Employee** ↔ **Department**：`1:N`（一个部门包含多名员工）。  
  - **Employee** ↔ **EmergencyContact**：`1:N`（一个员工可有多个紧急联系人）。  
  - **Employee** ↔ **UnitedWayContribution**：`1:N`（员工可多次调整捐款）。  
  - **Employee** ↔ **SavingsBond**：`1:N`（员工可购买多笔债券）。  
  - **SavingsBond** ↔ **BondOwner**：`1:1`（每笔债券对应一个所有者）。  
  - **Department** ↔ **Site**：`1:N`（一个站点包含多个部门）。  

---

### **3. 基于键的数据模型（Key-Based Data Model）**  
![](https://via.placeholder.com/800x400.png?text=Key-Based+Data+Model+Diagram)  
- **主键与外键设计**：  
  - **Employee**：`EmployeeID`（主键），`DepartmentID`（外键引用Department表）。  
  - **SavingsBond**：`BondID`（主键），`OwnerID`（外键引用BondOwner表）。  
  - **TelephoneListing**：`ListingID`（主键），`EmployeeID`（外键引用Employee表）。  

---

### **4. 完全属性化的数据模型（Fully Attributed Data Model）**  
![](https://via.placeholder.com/800x400.png?text=Fully+Attributed+Data+Model+Diagram)  
- **属性细化示例**：  
  - **Employee**：  
    ```sql  
    EmployeeID (PK), FirstName, LastName, MiddleInitial, SSN, BirthDate,  
    MaritalStatus, Address, Phone, DepartmentID (FK), HireDate  
    ```  
  - **SavingsBond**：  
    ```sql  
    BondID (PK), EmployeeID (FK), Denomination, DeductionType,  
    OwnerSSN, OwnerName, PurchaseDate  
    ```  
  - **UnitedWayContribution**：  
    ```sql  
    ContributionID (PK), EmployeeID (FK), DeductionType, Amount,  
    EffectiveStartDate, EffectiveEndDate  
    ```  

---

### **5. 高级选项：处理复杂雇佣状态**  
- **扩展模型设计**：  
  - **新增实体**：**EmploymentHistory**  
    ```sql  
    HistoryID (PK), EmployeeID (FK), EmploymentStatus (ENUM: Active/Retired/Terminated/Leave),  
    EmploymentType (ENUM: Full-Time/Part-Time), StartDate, EndDate  
    ```  
  - **关系调整**：  
    - **Employee** ↔ **EmploymentHistory**：`1:N`（一名员工可有多个雇佣记录）。  

---

### **6. 模型验证与一致性检查**  
- **数据完整性规则**：  
  - `EmployeeID`在所有相关表中唯一且不可为空。  
  - `SSN`在Employee和BondOwner表中需唯一。  
  - `EmploymentStatus`需符合预设枚举值（如Active、Retired等）。  
- **范式优化**：  
  - 所有模型满足第三范式（3NF），消除传递依赖。  

--- 

**总结**：通过上述数据模型设计，A-1 IS的ESSS系统可实现以下目标：  
1. **数据集中管理**：消除冗余，确保员工信息一致性。  
2. **灵活扩展**：支持多站点、多部门结构，适应公司增长需求。  
3. **自助服务支持**：员工可实时更新个人信息，减少HR人工干预。  
4. **战略目标对齐**：提供管理层监控工具（如参与度报告），支持United Way和储蓄债券目标。  

**交付物示例**（基于文本描述，需用建模工具生成图形化模型）：  
- **实体定义矩阵**（Excel或表格形式）。  
- **数据模型图**（使用工具如Lucidchart、ER/Studio或PowerDesigner绘制）。  
- **SQL脚本**（包含表结构、主键/外键、约束定义）。
