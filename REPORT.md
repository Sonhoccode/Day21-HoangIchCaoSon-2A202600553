# Lab 21 - Evaluation Report

**Hoc vien**: Hoang Ich Cao Son - 2A202600553  
**Ngay nop**: 2026-06-25  
**Submission option**: C - Code-only

## 1. Setup

- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`, lay 200 mau tieng Viet bang `shuffle(seed=42).select(range(200))`
- **Cot du lieu**: `instruction_vi`, `input_vi`, `output_vi`
- **Split**: 180 train / 20 eval, seed = 42
- **Token length**: min = 25, max = 738, p50 = 227, p95 = 562, p99 = 704
- **max_seq_length**: 1024, vi p95 = 562 va notebook T4 cap tai 1024
- **GPU**: Tesla T4, 14.563 GB VRAM
- **Training config**: 3 epochs, batch size = 1, gradient accumulation = 8, effective batch = 8, learning rate = 2e-4, cosine scheduler, optimizer `adamw_8bit`
- **LoRA target modules**: `["q_proj", "v_proj"]`
- **Training cost uoc tinh**: 12.2 phut, khoang `$0.07` voi gia T4 `$0.35/giờ`

## 2. Rank Experiment Results

| Rank | Alpha | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|---:|---:|---:|---:|---:|---:|---:|
| 8 | 16 | 1,843,200 | 4.00 min | 7.22 GB | 1.5577 | 4.7479 |
| 16 | 32 | 3,686,400 | 4.26 min | 6.62 GB | 1.5161 | 4.5544 |
| 64 | 128 | 14,745,600 | 3.99 min | 8.00 GB | 1.4768 | 4.3790 |

Ket qua chi tiet nam trong `results/rank_experiment_summary.csv`.

## 3. Loss Curve Analysis

Loss curve duoc luu tai `results/loss_curve.png`. Do chay tren T4, notebook tat evaluation giua training de tranh OOM, nen curve chu yeu la training loss theo step; eval loss chi duoc tinh sau khi adapter da save. Training loss giam kha deu trong 69 steps, khong co dau hieu bat on ro rang. Tuy nhien, vi eval set chi co 20 mau va khong log eval loss theo tung checkpoint, ket luan overfitting chi nen xem la tam thoi. Neu mo rong lab, minh se bat eval theo epoch tren GPU lon hon hoac tang eval set len 50-100 mau de quan sat train/eval gap tot hon.

## 4. Qualitative Comparison

| # | Prompt | Base | Fine-tuned r=16 | Nhan xet |
|---:|---|---|---|---|
| 1 | Giai thich machine learning cho nguoi moi bat dau. | Dinh nghia ML la mot phan cua AI, hoc tu du lieu de du doan/hanh dong. | Dien giai ML la cong nghe may tinh hoc va cai thien du doan tu du lieu, gan voi AI. | Fine-tuned mach lac va than thien hon. |
| 2 | Viet code Python tinh Fibonacci thu n. | Dua ra huong de quy/vong lap va code mau. | Dua ra ham `fibonacci(n)` co kiem tra `n < 0` va xu ly bien. | Fine-tuned co xu ly input ro hon. |
| 3 | Liet ke 5 nguyen tac UI/UX. | Cau tra loi dung nhung lap tu va dai. | Danh sach ngan hon, gom chuyen doi, thich ung, don gian. | Cai thien ve do gon, nhung noi dung chua that sau. |
| 4 | Tom tat LoRA va QLoRA. | Noi LoRA/QLoRA la low-rank va quantized LoRA, con hoi chung chung. | Goi sai LoRA thanh "Layer-wise Adaptive Regularization Optimization". | Fine-tuned bi degraded, day la loi thuat ngu nghiem trong. |
| 5 | Phan biet prompt engineering, RAG va fine-tuning. | Phan biet ca ba o muc khai quat. | Dien dat tu nhien hon ve prompt engineering, RAG va fine-tuning. | Fine-tuned tot hon ve van phong, nhung chua du sau. |

Bang CSV nam trong `results/qualitative_comparison.csv`. Mot diem quan trong la khong phai fine-tune luon cai thien moi prompt: vi dataset la Vietnamese Alpaca general, model hoc phong cach tra loi tieng Viet tot hon, nhung voi kien thuc chuyen nganh nhu LoRA/QLoRA van co the hallucinate.

## 5. Conclusion ve Rank Trade-off

Trong thi nghiem nay, rank 64 cho perplexity tot nhat: 4.3790, thap hon rank 16 la 4.5544 va rank 8 la 4.7479. Tuy nhien, chi phi tham so cua rank 64 cao hon rat nhieu: 14.7M trainable params, gap 4 lan rank 16 va 8 lan rank 8. Muc cai thien perplexity tu rank 8 len rank 16 la khoang 0.1935, con tu rank 16 len rank 64 la khoang 0.1754. Dieu nay cho thay rank cao hon van co loi, nhung loi ich bien giam dan so voi so tham so tang them.

Neu chon de deploy production cho bai toan Vietnamese instruction-following nho nay, minh se chon rank 16. Rank 16 co perplexity gan voi rank 64, trainable params vua phai, adapter nhe hon va phu hop hon neu can luu nhieu adapter cho nhieu domain. Rank 8 phu hop khi can train nhanh hoac tiet kiem dung luong toi da, nhung chat luong eval thap hon ro. Rank 64 chi nen dung khi dataset lon hon, domain phuc tap hon, hoac khi da xac minh bang eval chat luong cao rang phan cai thien nho ve perplexity that su tao gia tri cho nguoi dung.

## 6. What I Learned

- LoRA rank khong chi la tham so ky thuat; no la trade-off giua capacity, adapter size, VRAM va loi ich chat luong.
- Perplexity giup so sanh dinh luong, nhung qualitative evaluation van can thiet vi model co the giam loss ma van sai thuat ngu o prompt chuyen nganh.
- Fine-tuning phu hop de hoc style/format/phan phoi tra loi, nhung khong thay the RAG khi van de la kien thuc chinh xac hoac noi dung can cap nhat.
