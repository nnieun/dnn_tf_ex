
1단계. 나만의 작업 공간 만들기 (가상환경)
bash
uv init --bare

레고 블록을 조립할 때 다른 사람 블록이랑 안 섞이게 나만의 상자를 만드는 것과 비슷해. uv는 파이썬 프로그램들을 깔끔하게 관리해주는 도구야.

2단계. 그래픽카드가 잘 인식되는지 확인
bash
nvidia-smi

이건 "내 컴퓨터에 그래픽카드(GPU)가 잘 꽂혀 있고 작동하는지" 확인하는 명령어야. 마치 게임기 전원 켜지는지 체크하는 거랑 비슷해.

3단계. 필요한 도구들 설치

그래픽카드 모델에 따라 설치하는 TensorFlow 버전이 달라:

그래픽카드	설치 명령
RTX 4060 (최신)	uv add "tensorflow[and-cuda]" (2.21.0)
RTX 3060	uv add "tensorflow[and-cuda]==2.20.0"
GTX 1050 (오래된 카드)	uv add "tensorflow[and-cuda]==2.17.1"

그리고 데이터 분석·그래프 도구들도 같이 설치:

bash
uv add seaborn pandas matplotlib scikit-learn

비유: 그래픽카드는 "자동차 엔진"이고, TensorFlow 버전은 "그 엔진에 맞는 기름 종류"야. 오래된 엔진(GTX 1050)에 최신 기름(2.21.0)을 넣으면 안 맞을 수 있어서 버전을 맞춰주는 거지.

4단계. 주피터 노트북이랑 연결하기

주피터 노트북은 코드를 한 줄씩 실행하고 결과를 바로 보는 노트 같은 프로그램이야.

bash
uv add ipykernel
uv run python -m ipykernel install --user --name .venv

내가 만든 작업 상자(가상환경)를 주피터 노트북이 쓸 수 있게 등록하는 과정이야.

5단계. WSL2에서 GPU를 진짜로 쓸 수 있게 설정 (제일 중요!)

여기가 핵심이야. WSL2(윈도우 안에 있는 리눅스)는 그래픽카드 관련 파일 위치를 스스로 잘 못 찾을 때가 있어서, "여기 있으니까 찾아봐" 하고 알려주는 스크립트를 만들어야 해.

형이 알아낸 정답 방법 (이걸 써야 오류가 안 남):

bash
cat > run_gpu_code.sh <<'EOF'
#!/bin/bash
export LD_LIBRARY_PATH=$(find .venv/lib/python*/site-packages/nvidia \
  -type d -name lib | tr '\n' ':'):$LD_LIBRARY_PATH
uv run "$@"
EOF

이 명령어의 의미:

cat > 파일이름 <<'EOF' ... EOF → "EOF부터 EOF까지의 내용을 통째로 파일에 저장해줘"라는 뜻이야. 마치 편지지에 내용을 쭉 적고 봉투에 넣는 것처럼, 줄바꿈이 있는 긴 내용을 한 번에 파일로 만드는 방법이야.
원래 자료처럼 그냥 텍스트 편집기로 줄마다 따로 입력하면, 줄바꿈이나 특수문자가 깨져서 스크립트가 오류 나는 경우가 있어. cat <<'EOF' 방식은 내용을 통째로 정확하게 넣어주니까 훨씬 안전해.

실행 권한 주기:

bash
chmod +x run_gpu_code.sh

→ "이 파일은 프로그램처럼 실행해도 돼"라고 허락하는 명령어야.

6단계. 진짜로 GPU가 인식되는지 테스트
bash
./run_gpu_code.sh python -c "import tensorflow as tf; print('TensorFlow:', tf.__version__); print('GPU:', tf.config.list_physical_devices('GPU'))"

이 명령어는:

방금 만든 run_gpu_code.sh를 실행하고
그 안에서 파이썬으로 TensorFlow를 불러와서
"지금 TensorFlow 버전이 뭐야?" + "GPU가 보이니?"를 출력해줘

결과에 GPU: [PhysicalDevice(...)] 같은 게 나오면 성공! 빈 리스트 []면 GPU를 못 찾은 거야.

7단계. 훈련 중에 그래픽카드가 얼마나 일하는지 실시간으로 보기
bash
watch -n 1 nvidia-smi

1초마다 화면을 새로고침해서 그래픽카드 사용량(VRAM)을 계속 보여줘. 게임 할 때 성능 모니터 켜놓는 거랑 똑같아.

핵심 요약 (오류 안 나게 하는 포인트):
스크립트 파일(run_gpu_code.sh)을 만들 때는 줄 단위로 손으로 옮겨 적지 말고, cat > 파일 <<'EOF' ... EOF 통짜 방식으로 만들어야 줄바꿈/문법이 안 깨지고 정확하게 저장돼. 이게 형이 오류를 해결한 핵심 이유야.

