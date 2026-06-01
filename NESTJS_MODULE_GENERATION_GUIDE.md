# NestJS Module Generation Guide - Riphah UMS

Complete guide for generating and structuring all 17 NestJS modules with consistent patterns.

---

## Module Checklist (17 Modules)

### Completed ✅
- [x] Auth Module
- [x] Users Module  
- [x] Roles Module

### To Generate 🔄
- [ ] Admission Module
- [ ] Academic Module
- [ ] Examination Module
- [ ] Finance Module
- [ ] HR Module
- [ ] LMS Module
- [ ] Student Affairs Module
- [ ] Library Module
- [ ] Hostel Module
- [ ] Transport Module
- [ ] Research Module
- [ ] QEC Module
- [ ] Alumni Module
- [ ] Analytics Module
- [ ] Notification Module
- [ ] Integration Module

---

## Module Structure Template

Each module follows this standard structure:

```
src/
├── {module}.module.ts
├── {module}.service.ts
├── {module}.controller.ts
├── dto/
│   ├── create-{entity}.dto.ts
│   ├── update-{entity}.dto.ts
│   └── {entity}-filter.dto.ts
├── entities/
│   └── {entity}.entity.ts
├── interceptors/
│   └── {module}.interceptor.ts
└── decorators/
    └── {module}.decorator.ts
```

---

## 1. ADMISSION MODULE

**Purpose**: Lead tracking, applications, merit lists, offers, enrollments  
**Database Models**: Lead, Application, ApplicationDocument, MeritRank, AdmissionOffer, FeeVoucher  
**Key Workflows**: Lead → Application → Test → Merit → Offer → Enrollment

### Controllers

```typescript
// admission.controller.ts
@Controller('api/admission')
@UseGuards(AuthGuard('jwt'), RolesGuard)
export class AdmissionController {
  constructor(private readonly admissionService: AdmissionService) {}

  // Lead Management
  @Post('leads')
  @Roles('Admissions Officer')
  createLead(@Body() dto: CreateLeadDto) {
    return this.admissionService.createLead(dto);
  }

  @Get('leads')
  @Roles('Admissions Officer', 'Registrar')
  getLeads(@Query() filterDto: LeadFilterDto) {
    return this.admissionService.getLeads(filterDto);
  }

  @Get('leads/:id')
  @Roles('Admissions Officer', 'Registrar', 'Student')
  getLeadById(@Param('id') id: string) {
    return this.admissionService.getLeadById(id);
  }

  // Application Management
  @Post('applications')
  @Roles('Student', 'Admissions Officer')
  createApplication(@Body() dto: CreateApplicationDto) {
    return this.admissionService.createApplication(dto);
  }

  @Get('applications')
  @Roles('Admissions Officer', 'Registrar')
  getApplications(@Query() filterDto: ApplicationFilterDto) {
    return this.admissionService.getApplications(filterDto);
  }

  @Patch('applications/:id/status')
  @Roles('Admissions Officer')
  updateApplicationStatus(
    @Param('id') id: string,
    @Body() dto: UpdateApplicationStatusDto
  ) {
    return this.admissionService.updateApplicationStatus(id, dto);
  }

  // Merit Lists
  @Post('merit-lists')
  @Roles('Admissions Officer')
  generateMeritList(@Body() dto: GenerateMeritListDto) {
    return this.admissionService.generateMeritList(dto);
  }

  @Get('merit-lists/:programId')
  @Roles('Admissions Officer', 'Registrar', 'Student')
  getMeritList(@Param('programId') programId: string) {
    return this.admissionService.getMeritList(programId);
  }

  // Admission Offers
  @Post('offers')
  @Roles('Admissions Officer')
  generateOffers(@Body() dto: GenerateOffersDto) {
    return this.admissionService.generateOffers(dto);
  }

  @Patch('offers/:id/accept')
  @Roles('Student')
  acceptOffer(@Param('id') id: string) {
    return this.admissionService.acceptOffer(id);
  }

  @Patch('offers/:id/reject')
  @Roles('Student')
  rejectOffer(@Param('id') id: string) {
    return this.admissionService.rejectOffer(id);
  }

  // Fee Vouchers
  @Get('vouchers/:id')
  @Roles('Student', 'Finance Officer')
  getVoucher(@Param('id') id: string) {
    return this.admissionService.getVoucher(id);
  }
}
```

### Service

```typescript
// admission.service.ts
@Injectable()
export class AdmissionService {
  constructor(
    private prisma: PrismaService,
    private notificationService: NotificationService
  ) {}

  async createLead(dto: CreateLeadDto) {
    const lead = await this.prisma.lead.create({
      data: {
        ...dto,
        status: 'NEW'
      }
    });

    await this.notificationService.send({
      userId: 'admin',
      title: 'New Lead',
      message: `New admission lead: ${dto.firstName} ${dto.lastName}`,
      channel: 'PORTAL'
    });

    return lead;
  }

  async getApplications(filterDto: ApplicationFilterDto) {
    return this.prisma.application.findMany({
      where: {
        status: filterDto.status,
        lead: {
          firstName: { contains: filterDto.search || '' }
        }
      },
      include: {
        lead: true,
        program: true,
        documents: true,
        merits: true
      },
      skip: filterDto.skip,
      take: filterDto.take
    });
  }

  async generateMeritList(dto: GenerateMeritListDto) {
    // Fetch eligible applications
    const applications = await this.prisma.application.findMany({
      where: {
        programId: dto.programId,
        status: 'UNDER_REVIEW'
      }
    });

    // Calculate scores and rank
    const ranked = applications
      .map(app => ({
        ...app,
        score: (app.testScore || 0) * 0.6 + (app.interviewScore || 0) * 0.4
      }))
      .sort((a, b) => b.score - a.score)
      .map((app, index) => ({
        ...app,
        rank: index + 1
      }));

    // Save merit ranks
    const merits = await Promise.all(
      ranked.map(app =>
        this.prisma.meritRank.create({
          data: {
            applicationId: app.id,
            rank: app.rank,
            score: app.score,
            status: 'ELIGIBLE'
          }
        })
      )
    );

    return merits;
  }

  async acceptOffer(offerId: string) {
    const offer = await this.prisma.admissionOffer.findUnique({
      where: { id: offerId },
      include: { merit: { include: { application: true } } }
    });

    if (!offer) throw new NotFoundException('Offer not found');
    if (offer.status !== 'PENDING') throw new BadRequestException('Offer already processed');

    // Update offer status
    const updated = await this.prisma.admissionOffer.update({
      where: { id: offerId },
      data: { status: 'ACCEPTED' }
    });

    // Create fee voucher
    const voucher = await this.prisma.feeVoucher.create({
      data: {
        offerId: offerId,
        voucherNo: `FV-${Date.now()}`,
        amount: 150000, // Placeholder - fetch from fee structure
        semester: 1,
        dueDate: new Date(Date.now() + 15 * 24 * 60 * 60 * 1000), // 15 days
        status: 'ISSUED'
      }
    });

    return { offer: updated, voucher };
  }
}
```

### DTO

```typescript
// dto/create-lead.dto.ts
export class CreateLeadDto {
  @IsString()
  firstName: string;

  @IsString()
  lastName: string;

  @IsEmail()
  email: string;

  @IsPhoneNumber()
  phone: string;

  @IsString()
  program: string;

  @IsEnum(['WEBSITE', 'REFERRAL', 'ADVERTISEMENT', 'SOCIAL_MEDIA', 'EVENT', 'OTHER'])
  source: string;
}
```

### Module

```typescript
// admission.module.ts
@Module({
  controllers: [AdmissionController],
  providers: [AdmissionService],
  exports: [AdmissionService],
  imports: [NotificationModule]
})
export class AdmissionModule {}
```

---

## 2. ACADEMIC MODULE

**Purpose**: Courses, programs, sections, registrations, timetables  
**Database Models**: Program, Specialization, Course, Section, StudentRegistration, Grade  
**Key Endpoints**: Programs, Courses, Sections, Registrations, Timetables

### Core Endpoints

```
GET    /api/academic/programs
POST   /api/academic/programs
GET    /api/academic/programs/:id
PATCH  /api/academic/programs/:id

GET    /api/academic/courses
GET    /api/academic/courses/:id

GET    /api/academic/sections
POST   /api/academic/registrations
GET    /api/academic/registrations
PATCH  /api/academic/registrations/:id/status

GET    /api/academic/timetables
POST   /api/academic/timetables/generate
```

---

## 3. EXAMINATION MODULE

**Purpose**: Date sheets, seating plans, results, transcripts  
**Database Models**: Exam, ExamRoom, SeatingPlan, ExamAttendance, Grade  
**Key Workflows**: Schedule → Seating → Conduct → Results → Transcript

### Core Endpoints

```
POST   /api/examination/date-sheets
GET    /api/examination/date-sheets
POST   /api/examination/seating-plans/generate
GET    /api/examination/seating-plans
POST   /api/examination/attendance
GET    /api/examination/results
GET    /api/examination/transcripts/:studentId
```

---

## 4. FINANCE MODULE

**Purpose**: Fee structure, payments, scholarships, defaults  
**Database Models**: StudentFee, Payment, Scholarship, FeeVoucher  
**Key Workflows**: Fee Calculation → Voucher → Payment → Receipt

### Core Endpoints

```
GET    /api/finance/fee-structure
GET    /api/finance/student/:id/fees
POST   /api/finance/payments
GET    /api/finance/payments
PATCH  /api/finance/payments/:id/verify
GET    /api/finance/scholarships
POST   /api/finance/scholarships
```

---

## 5. HR MODULE

**Purpose**: Employee records, leave, attendance, payroll  
**Database Models**: Staff, FacultyMember, Leave, Salary, Performance  
**Key Workflows**: Attendance → Leave Request → Approval → Payroll

### Core Endpoints

```
GET    /api/hr/staff
GET    /api/hr/staff/:id
POST   /api/hr/leave/request
GET    /api/hr/leave/requests
PATCH  /api/hr/leave/requests/:id/approve
GET    /api/hr/attendance
POST   /api/hr/attendance/mark
GET    /api/hr/payroll
```

---

## 6. LMS MODULE

**Purpose**: Courses, lessons, assignments, gradebook  
**Database Models**: LmsCourse, Lesson, LmsAssignment, LmsSubmission, LmsEnrollment  
**Key Workflows**: Enrollment → Lesson → Assignment → Submission → Grading

### Core Endpoints

```
GET    /api/lms/courses
POST   /api/lms/courses
GET    /api/lms/courses/:id/lessons
POST   /api/lms/assignments
GET    /api/lms/assignments/:id
POST   /api/lms/assignments/:id/submissions
GET    /api/lms/gradebook/:studentId
```

---

## 7-17. OTHER MODULES

Similar structure for:

- **Student Affairs**: Complaints, events, counseling
- **Library**: Catalog, borrowing, returns  
- **Hostel**: Room allocation, fees, attendance
- **Transport**: Routes, buses, passes
- **Research**: Projects, publications
- **QEC**: Surveys, evaluations
- **Alumni**: Profiles, networking
- **Analytics**: Dashboards, reports
- **Notification**: Email, SMS, Push
- **Integration**: Third-party APIs

---

## Standard DTO Pattern

```typescript
export class CreateEntityDto {
  // Required fields
  @IsString()
  @IsNotEmpty()
  name: string;

  // Optional fields
  @IsOptional()
  @IsString()
  description?: string;

  // Validation decorators
  @IsEmail()
  email: string;

  @IsEnum(EntityStatus)
  status?: EntityStatus;

  @IsInt()
  @Min(0)
  quantity?: number;
}

export class UpdateEntityDto extends PartialType(CreateEntityDto) {}

export class EntityFilterDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsEnum(EntityStatus)
  status?: EntityStatus;

  @IsOptional()
  @IsInt()
  skip?: number = 0;

  @IsOptional()
  @IsInt()
  take?: number = 10;
}
```

---

## Standard Service Pattern

```typescript
@Injectable()
export class EntityService {
  constructor(
    private prisma: PrismaService,
    private logger: LoggerService
  ) {}

  async create(dto: CreateEntityDto) {
    this.logger.log(`Creating entity: ${JSON.stringify(dto)}`);
    
    const entity = await this.prisma.entity.create({
      data: dto,
      include: { /* relations */ }
    });

    this.logger.log(`Entity created with ID: ${entity.id}`);
    return entity;
  }

  async findAll(filterDto: EntityFilterDto) {
    return this.prisma.entity.findMany({
      where: { /* filters */ },
      skip: filterDto.skip,
      take: filterDto.take,
      orderBy: { createdAt: 'desc' }
    });
  }

  async findOne(id: string) {
    const entity = await this.prisma.entity.findUnique({
      where: { id },
      include: { /* relations */ }
    });

    if (!entity) {
      throw new NotFoundException(`Entity with ID ${id} not found`);
    }

    return entity;
  }

  async update(id: string, dto: UpdateEntityDto) {
    return this.prisma.entity.update({
      where: { id },
      data: dto,
      include: { /* relations */ }
    });
  }

  async remove(id: string) {
    return this.prisma.entity.delete({
      where: { id }
    });
  }
}
```

---

## Standard Controller Pattern

```typescript
@Controller('api/entities')
@UseGuards(AuthGuard('jwt'), RolesGuard)
@ApiBearerAuth()
export class EntityController {
  constructor(private readonly entityService: EntityService) {}

  @Post()
  @Roles('Admin', 'Manager')
  @ApiOperation({ summary: 'Create entity' })
  @ApiResponse({ status: 201, description: 'Entity created' })
  create(@Body() dto: CreateEntityDto) {
    return this.entityService.create(dto);
  }

  @Get()
  @Roles('Admin', 'Manager', 'User')
  @ApiOperation({ summary: 'List entities' })
  findAll(@Query() filterDto: EntityFilterDto) {
    return this.entityService.findAll(filterDto);
  }

  @Get(':id')
  @Roles('Admin', 'Manager', 'User')
  findOne(@Param('id') id: string) {
    return this.entityService.findOne(id);
  }

  @Patch(':id')
  @Roles('Admin', 'Manager')
  update(@Param('id') id: string, @Body() dto: UpdateEntityDto) {
    return this.entityService.update(id, dto);
  }

  @Delete(':id')
  @Roles('Admin')
  remove(@Param('id') id: string) {
    return this.entityService.remove(id);
  }
}
```

---

## Module Registration Pattern

```typescript
@Module({
  controllers: [EntityController],
  providers: [EntityService],
  exports: [EntityService],
  imports: [/* other modules */]
})
export class EntityModule {}
```

Then import in `app.module.ts`:

```typescript
@Module({
  imports: [
    ConfigModule,
    PrismaModule,
    AuthModule,
    UsersModule,
    RolesModule,
    AdmissionModule,      // <- Add here
    AcademicModule,       // <- Add here
    ExaminationModule,    // <- Add here
    FinanceModule,        // <- Add here
    // ... etc
  ]
})
export class AppModule {}
```

---

## Generation Commands

### Create a new module scaffold:

```bash
nest g mo {module-name}
nest g co {module-name}
nest g s {module-name}
```

### Example:

```bash
# Generate Admission module
nest g mo admission
nest g co admission
nest g s admission

# Add DTOs
mkdir src/admission/dto
touch src/admission/dto/create-lead.dto.ts
touch src/admission/dto/create-application.dto.ts
```

---

## Next Steps

1. **Fix Prisma Schema** - Resolve syntax errors in `schema.prisma`
2. **Run Migration** - `npm run prisma:migrate -- --name initial`
3. **Generate Modules** - Use CLI commands above for each module
4. **Implement Services** - Follow patterns above for each module
5. **Add Tests** - Create unit and e2e tests
6. **Document APIs** - Swagger documentation auto-generated
7. **Deploy** - Docker and deployment configuration

---

**Status**: Guide Complete  
**Target Completion**: 20 modules with consistent architecture and patterns
