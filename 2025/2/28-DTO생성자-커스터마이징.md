- 생성자 초기화 옵션 커스터마이징
   - trade-off: OCP vs. Validation 통과하여 최종 생성시에는 널 방지.
   - 클래스 관점에서는 OCP 위반. 시스템 관점에서는 OCP 지키며 다양한 생성자 조건 이용해 DTO생성 가능.    

```java
  //DTO Initializer
    public static final class CreationInitializer {

        public static final String CHECK_IN_TITLE = "체크인 요청";

        public static String createTitle(String placeName) {
            final String SPACE = " ";
            return placeName + SPACE + CHECK_IN_TITLE;
        }

        public static LocalDateTime calculateEnd(LocalDateTime start, Integer option) {
            try {
                return start.plusMinutes(option);
            } catch (Exception e) {
                throw new InfraException("체크인 요청 시간 계산 중 오류가 발생했습니다.", e);
            }
        }
    }

    // DTO - Request
    @Getter
    @Validated
    public static final class Creation implements HelpDetail.DTO, Progress.DTO {

        @NotNull(message = NO_CHECK_IN_REGISTER_ID)
        private final Long helpRegisterId;

        @NotNull(message = NO_PLACE_ID)
        private final String placeId;

        @NotNull(message = NO_PLACE_NAME)
        private final String placeName;

        @NotNull(message = NO_START)
        private final LocalDateTime start;

        @NotNull(message = NO_TIME_OPTION)
        private final Integer option;

        @NotNull(message = NO_REWARD)
        private final Long reward;

        @NotNull(message = NO_PLACE_NAME)
        private String title;

        @NotNull(message = NO_START_AND_TIME_OPTION)
        @Setter(AccessLevel.PRIVATE)
        private LocalDateTime end;

        @Nullable
        private final Long helperId;

        @Nullable
        private final String photoPath;

        @JsonIgnore
        @NonNull
        private final Progress.ProgressStatus status;

        private final boolean completed;

        @Builder
        @Jacksonized
        public Creation(Long helpRegisterId, String placeId, String placeName, LocalDateTime start, Integer option, Long reward) {
            this.helpRegisterId = helpRegisterId;
            this.placeId = placeId;
            this.placeName = placeName;
            this.start = start;
            this.option = option;
            this.reward = reward;

            //커스터마이징
            boolean willFailTitleRelatedValidation = placeName == null;
            boolean willFailEndRelatedValidation = (start == null || option == null);
            boolean willFailValidation = willFailTitleRelatedValidation || willFailEndRelatedValidation;
            if (!willFailValidation) {
                this.title = Objects.requireNonNull(CreationInitializer.createTitle(placeName));
                this.end = Objects.requireNonNull(CreationInitializer.calculateEnd(start, option));
            }

            final Long NO_HELPER_AT_CREATED = null;
            final String NO_PHOTO_AUTHENTICATION_AT_CREATED = null;
            this.helperId = NO_HELPER_AT_CREATED;
            this.photoPath = NO_PHOTO_AUTHENTICATION_AT_CREATED;
            this.status = Objects.requireNonNull(new Progress.Created());
            this.completed = Objects.requireNonNull(false);
        }

        public Optional<Long> getHelperId() {
            return Optional.ofNullable(helperId);
        }

        public Optional<String> getPhotoPath() {
            return Optional.ofNullable(photoPath);
        }
    }

```
